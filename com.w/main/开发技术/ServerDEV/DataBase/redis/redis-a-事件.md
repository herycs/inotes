# 事件

> redis是事件驱动程序，主要以下两类事件
>
> - 文件事件：Redis使用套接字与客户端连接，文件事件就是对套接字的抽象
> - 时间事件：Redis中需要在指定时间执行的事件

# 文件事件

基于Reactor模式开发网络事件处理器（file event handler）

- 文件事件基于IO多路复用监听多个套接字，并依据套接字目前任务为其关联事件处理器
- 当操作套接字时会触发事件，这是就会用到上述的事件处理器

## 文件事件处理器构成

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721195326757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

### 多路复用器实现

> redis 中基于对已有的复用器包装实现，提供了相同的API，故底层的多路复用器可替换
>
> ae_epoll.c
>
> ae_evport.c
>
> ae_kqueue.c
>
> ae_select.c

```c
#ifdef HAVE_EVPORT
#include "ae_evport.c"
#else
    #ifdef HAVE_EPOLL
    #include "ae_epoll.c"
    #else
        #ifdef HAVE_KQUEUE
        #include "ae_kqueue.c"
        #else
        #include "ae_select.c"
        #endif
    #endif
#endif
```

上述展示了文件导入的策略，也是对多路复用器选择的策略，依次为evport -> epoll -> kqueue -> select，性能依次降低

## 事件类型

- AE_READABLE

    套接字可读，或者新的可应答套接字出现

- AE_WRITABLE

    套接字可写

给定套接字加入多路复用器监听范围，并对事件和时间处理器进行关联

```c
int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask, aeFileProc *proc, void *clientData) {
    if (fd >= eventLoop->setsize) {
        errno = ERANGE;
        return AE_ERR;
    }
    aeFileEvent *fe = &eventLoop->events[fd];

    if (aeApiAddEvent(eventLoop, fd, mask) == -1)
        return AE_ERR;
    fe->mask |= mask;
    if (mask & AE_READABLE) fe->rfileProc = proc;
    if (mask & AE_WRITABLE) fe->wfileProc = proc;
    fe->clientData = clientData;
    if (fd > eventLoop->maxfd)
        eventLoop->maxfd = fd;
    return AE_OK;
}
```

取消监听，取消关联

```c
void aeDeleteFileEvent(aeEventLoop *eventLoop, int fd, int mask)
{
    if (fd >= eventLoop->setsize) return;
    aeFileEvent *fe = &eventLoop->events[fd];
    if (fe->mask == AE_NONE) return;

    /* We want to always remove AE_BARRIER if set when AE_WRITABLE
     * is removed. */
    if (mask & AE_WRITABLE) mask |= AE_BARRIER;

    aeApiDelEvent(eventLoop, fd, mask);
    fe->mask = fe->mask & (~mask);
    if (fd == eventLoop->maxfd && fe->mask == AE_NONE) {
        /* Update the max fd */
        int j;

        for (j = eventLoop->maxfd-1; j >= 0; j--)
            if (eventLoop->events[j].mask != AE_NONE) break;
        eventLoop->maxfd = j;
    }
}
```

获取指定套接字正在被监听的事件类型

```c
int aeGetFileEvents(aeEventLoop *eventLoop, int fd) {
    if (fd >= eventLoop->setsize) return 0;
    aeFileEvent *fe = &eventLoop->events[fd];

    return fe->mask;
}
```

返回值：

- AE_NONE
- AE_READABLE
- AE_WRITABLE
- 读和写正在被监听：AE_READABLE | AE_WRITABLE

## 处理器

### 连接应答处理器

初始化时会将这个处理器和服务器监听套接字的AE_REAEABLE关联

### 命令请求处理器

### 命令回复处理器

一次请求处理示例：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721201244520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

# 时间事件

> - 定时事件
> - 周期事件

组成：id + when(UNIX时间戳) + timeProc(时间事件处理器)

## 实现

无序链表，不按照when排序，执行时需要遍历整个链表

- 正常模式下只允许一个serverCron
- benchmark模式下允许两个

以事件处理角度看Redis允许流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721204148694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

事件调度执行规则

1. 调度函数阻塞时间由到达时间最接近当前时间的时间事件决定，避免对时间事件进行频繁轮询
2. 文件事件随机出现，不断的处理文件事件，时间会趋于时间事件的到达时间
3. 文件事件和时间事件都是同步，有序，原子，不存在抢占，也不会中断事件处理，尽可能减少阻塞时间，必要时事件可让出执行权
4. 时间事件在文件事件之后执行，故执行时间可能比时间事件设定的到达时间稍晚一点
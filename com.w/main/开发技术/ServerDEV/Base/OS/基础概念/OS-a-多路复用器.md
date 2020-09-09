# 多路复用器

## select

### 认识

select 只有一个系统调用，每次要监听都要将其从用户态传到内核，有事件时返回整个集合。

### 函数

早期，linux中采用轮询fd的方式

```shell
-- linux/posix_types.h文件中对 select 大小修饰

# define __FD_SETSIZE  1024
```

## epoll

### 认识

内核 2.6 提出

epoll 监听的 fd（file descriptor）集合是常驻内核的，它有 3 个系统调用 （*epoll_create*, *epoll_wait*, *epoll_ctl*），通过 *epoll_wait* 可以多次监听同一个 fd 集合，只返回可读写那部分

### 函数

创建一个epoll句柄，创建fd个数，size个，epoll本身会占用一个，总计（size + 1)个

```c
int epoll_create(int size);
```

epoll的事件注册函数，它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。

```c
/*
	1. epfd 创建函数返回值，epoll_create()
	2. op 动作
		EPOLL_CTL_ADD：注册新的fd到epfd中；
		EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
		EPOLL_CTL_DEL：从epfd中删除一个fd；
	3. fd 监听的fd
	4. event 
		EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
        EPOLLOUT：表示对应的文件描述符可以写；
        EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
        EPOLLERR：表示对应的文件描述符发生错误；
        EPOLLHUP：表示对应的文件描述符被挂断；
        EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
        EPOLLONESHOT：只监听一次事件
*/
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

等待事件产生

```c
/*
参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个 maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。
*/
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

## 触发模式

### 水平触发(level-trggered)

LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket

只要文件描述符关联的读内核缓冲区非空，有数据可以读取，就一直发出可读信号进行通知，
 当文件描述符关联的内核写缓冲区不满，有空间可以写入，就一直发出可写信号进行通知
 LT模式支持阻塞和非阻塞两种方式。epoll默认的模式是LT。

<u>水平触发是只要数据未处理完，就会一直发送信号</u>

### 边缘触发(edge-triggered)

当文件描述符关联的读内核缓冲区由空转化为非空的时候，则发出可读信号进行通知，
当文件描述符关联的内核写缓冲区由满转化为不满的时候，则发出可写信号进行通知

<u>边缘触发仅仅在空变为非空的时候通知一次</u>

ET模式仅当状态发生变化的时候才获得通知，如果要采用ET模式,需要一直read/write直到出错为止,很多人反映为什么采用ET模式只接收了一部分数据就再也得不到通知了,大多因为这样


# RDB持久化

> rdb命令生成压缩的二进制文件

## 文件创建和导入

### 文件创建

```c
save | bgsave
```

实现

```c
int rdbSave(char *filename, rdbSaveInfo *rsi) {
    char tmpfile[256];
    char cwd[MAXPATHLEN]; /* Current working dir path for error messages. */
    FILE *fp;
    rio rdb;
    int error = 0;

    snprintf(tmpfile,256,"temp-%d.rdb", (int) getpid());
    fp = fopen(tmpfile,"w");
    //...
}
```

```c
int rdbSaveBackground(char *filename, rdbSaveInfo *rsi) {
    pid_t childpid;

    if (hasActiveChildProcess()) return C_ERR;

    server.dirty_before_bgsave = server.dirty;
    server.lastbgsave_try = time(NULL);
    openChildInfoPipe();

    if ((childpid = redisFork()) == 0) {
        int retval;

        /* Child */
        redisSetProcTitle("redis-rdb-bgsave");
        redisSetCpuAffinity(server.bgsave_cpulist);
        retval = rdbSave(filename,rsi);
        if (retval == C_OK) {
            sendChildCOWInfo(CHILD_INFO_TYPE_RDB, "RDB");
        }
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        /* Parent */
        if (childpid == -1) {
            closeChildInfoPipe();
            server.lastbgsave_status = C_ERR;
            serverLog(LL_WARNING,"Can't save in background: fork: %s",
                strerror(errno));
            return C_ERR;
        }
        serverLog(LL_NOTICE,"Background saving started by pid %d",childpid);
        server.rdb_save_time_start = time(NULL);
        server.rdb_child_pid = childpid;
        server.rdb_child_type = RDB_CHILD_TYPE_DISK;
        return C_OK;
    }
    return C_OK; /* unreached */
}
```

### 载入

RDB文件的载入在服务器启动时执行

实现

```c
int rdbLoad(char *filename, rdbSaveInfo *rsi, int rdbflags) {
    FILE *fp;
    rio rdb;
    int retval;

    if ((fp = fopen(filename,"r")) == NULL) return C_ERR;
    startLoadingFile(fp, filename,rdbflags);
    rioInitWithFile(&rdb,fp);
    retval = rdbLoadRio(&rdb,rdbflags,rsi);
    fclose(fp);
    stopLoading(retval==C_OK);
    return retval;
}
```

### AOFvsRDB

- 服务器开启了AOF持久化功能，优先使用AOF恢复
- AOF关闭，使用RDB还原数据库

```c
void loadDataFromDisk(void) {
    long long start = ustime();
    if (server.aof_state == AOF_ON) {//AOF开启，使用AOF实现载入
        if (loadAppendOnlyFile(server.aof_filename) == C_OK)
            serverLog(LL_NOTICE,"DB loaded from append only file: %.3f seconds",(float)(ustime()-start)/1000000);
    } else {//未开启AOF使用RDB载入
        rdbSaveInfo rsi = RDB_SAVE_INFO_INIT;
        if (rdbLoad(server.rdb_filename,&rsi,RDBFLAGS_NONE) == C_OK) {
            serverLog(LL_NOTICE,"DB loaded from disk: %.3f seconds", (float)(ustime()-start)/1000000);
            //...
        }
    }
}
```

当服务器处于BGSAVE状态下时，客户端发送的bgsave，save命令会被拒绝

bgsave和bgrewriteaof不能同时执行（主要是考虑到性能，因为都是由子进程执行操作，两个子进程并发进行磁盘写入在性能方面的影响不容小觑）

- bgsave在执行，bgrewriteaof会延迟到bgsave执行完毕再执行
- bgrewriteaof在执行，bgsave命令直接拒绝

## 服务状态

RDB载入状态，服务不可用（阻塞）

## 间隔存储

配置 save time1 times

例如：

```c
save 500 2	//500s内至少执行2次刷新,满足条件时bgsave会执行
```

## Dirty计数器&lastsave

> dirty计数器：上次持久化之后，数据库执行了多少次修改
>
> lastsave：UNIX时间戳，上一次成功执行save | bgsave时间

```c
// server.h
struct redisServer {
    long long dirty;                /* Changes to DB from the last save */
    time_t lastsave;                /* Unix time of last successful save */
}
```

server定时器触发执行时会检查所有许可保存的条件，满足则会调用bgsave执行保存操作

```c
//默认每百秒触发一次
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
if (hasActiveChildProcess() || ldbPendingChildren())
    {
        checkChildrenDone();
    } else {
        for (j = 0; j < server.saveparamslen; j++) {
            struct saveparam *sp = server.saveparams+j;
            //遍历所有保存条件，符合则继续保存
            if (server.dirty >= sp->changes &&
                server.unixtime-server.lastsave > sp->seconds &&
                (server.unixtime-server.lastbgsave_try >
                 CONFIG_BGSAVE_RETRY_DELAY ||
                 server.lastbgsave_status == C_OK))
            {
                serverLog(LL_NOTICE,"%d changes in %d seconds. Saving...",
                    sp->changes, (int)sp->seconds);
                rdbSaveInfo rsi, *rsiptr;
                rsiptr = rdbPopulateSaveInfo(&rsi);
                rdbSaveBackground(server.rdb_filename,rsiptr);
                break;
            }
        }
    //...
}
```

调用call会修改dirty计数器的值

```c
void call(client *c, int flags) {
    //...
    dirty = server.dirty;
    //...
    dirty = server.dirty-dirty;
    if (dirty < 0) dirty = 0;
}    
```

## 文件结构

REDIS + db_version + databases + EOF + check_sum

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721195412429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

# AOF持久化

当提出修改请求时，会在AOFbuffer末尾追加命令

- 执行set命令
- 执行rpush key value

实现

```c
struct redisServer {
    //...
    sds aof_buf;      /* AOF buffer, written before entering the event loop */
    //...
}    
```

## 写入与同步

Redis进程就是时间循环，当服务器每次结束一个事件循环前，都会调用flushAppendOnlyFile判断是否写入

判断依据是appendfsync选项的值

- always：每个事件循环执行完成，同步一次
- everysec：每秒一次写入+同步
- no：只管写入，同步交给操作系统

## 载入

执行过程：

1. 创建一个不带网络连接的伪客户端
2. AOF中分析并取出一条命令
3. 执行命令
4. 重复2,3直到文件结束

## 重写

解决文件膨胀问题，创建新的AOF文件，且新文件中不包含浪费空间的冗余命令

## 后台重写

使用子进程实现重写

- 子进程写入不影响父进程处理请求
- 子进程拥有副本，不需要加锁

重写期间命令执行过程

1. 执行命令
2. 执行后的写命令追加到AOF缓存区
3. 执行后的写命令追加到AOF重写缓冲区






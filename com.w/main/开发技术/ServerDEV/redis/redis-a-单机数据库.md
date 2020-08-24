# 数据库

## 基本定义

服务端定义

> server.h

```c
struct redisServer {
    pid_t pid;                  /* Main process pid. */
    redisDb *db;			//数组，保存着数据库中的所有数据库
    int dbnum;				//初始化服务器时创建多少数据库，默认16
    //...
}
```

客户端定义

```c
typedef struct client {
    uint64_t id;            /* Client incremental unique ID. */
    redisDb *db;            //Pointer to currently SELECTed DB,指向目标数据库
    //...
}
```

## 切换数据库

> redis客户端有要操作的目标数据库，默认情况下是0号数据库

redisDb* 的指针指向了当前的目标数据库

```c
127.0.0.1:6379> set demo1 123
OK
127.0.0.1:6379> get demo1
"123"
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> get demo1
(nil)
127.0.0.1:6379[2]> 
```

## 键空间

数据库结构

```c
typedef struct redisDb {
    dict *dict;                 // The keyspace for this DB,保存所有键值对
    dict *expires;              /* Timeout of keys with a timeout set */
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
    dict *ready_keys;           /* Blocked keys that received a PUSH */
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */
    int id;                     /* Database ID */
    long long avg_ttl;          /* Average TTL, just for stats */
    unsigned long expires_cursor; /* Cursor of the active expire cycle. */
    list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */
} redisDb;
```

键空间的键就是数据库的键，每个键都是个字符串对象

键空间的值和数据库的值对应，每个值可以是{字符串对象 | 列表对象 | 哈希表对象 | 集合对象 | 有序集合对象}

## CRUD原理

C

增加新键，也就是将键值对添加到键空间字典里面，键为字符串对象，值为redis对象

D

删除键，也就是在从键空间中删除键值对对象

U

更新键，对键空间中的键对应的值对象进行更新

R

对键取值，依据键从键空间取值

其他

对于数据库本身的命令，部分也在键空间完成

- FLUSHED
- RANDOMKEY
- EXISTS
- RENAME
- KEYS
- ...

## 读写键空间的维护

### 键空间

读取键后服务器会依据键存在与否更新键空间的{命中次数 | 未命中次数}

```c
info stats keyspace_hits
info stats keyspace_misses
```

### 键

读取一个键后服务器会更新键的LRU

若读取键时，发现键过期，则会删除这个键，再后续操作

客户端使用watch命令监视某个键

- 当这个键被修改后服务器会标记这个键为脏dirty，便于客户端观察到这种变化，每修改一次脏值+1
- 这个值会触发服务器的持久化&复制

若开启了数据库通知，键修改后服务器会按配置发送响应的数据库通知

#### 键过期时间

- setnx
- expire | expireat，单位s，xxxat设置为指定秒数的时间戳
- pexpire | pexpireat，单位ms

查看键的剩余生命

ttl | pttl

#### 保存过期时间

```c
typedef struct redisDb {
    //...
    dict *expires;              // Timeout of keys with a timeout set，这个字典保存了当前键空间所有键的过期时间
	//...
} redisDb;
```

#### 过期键判定

- 检查给定键是否存在过期字典，若存在，取得键的过期时间
- 检查当前时间是否大于过期时间，若是，则键已过期，否则键未注册

## 过期键删除策略

### 定时删除

- 定时器到了规定时间删除键
- 对内存友好，但是会占用CPU

### 惰性删除（被动）

实现

所有读写数据库的命令在实际执行前都会调用expireIfNeeded

```c
int expireIfNeeded(redisDb *db, robj *key) {
    if (!keyIsExpired(db,key)) return 0;
    if (server.masterhost != NULL) return 1;
    /* Delete the key */
    server.stat_expiredkeys++;
    propagateExpire(db,key,server.lazyfree_lazy_expire);
    notifyKeyspaceEvent(NOTIFY_EXPIRED, "expired",key,db->id);
    int retval = server.lazyfree_lazy_expire ? dbAsyncDelete(db,key) : dbSyncDelete(db,key);
    if (retval) signalModifiedKey(NULL,db,key);
    return retval;
}
```

- 获取时判定是否过期，过期则删除
- 对内存不友好，对CPU友好

### 定期删除

- 一定时间间隔检查一遍数据库，删除过期键
- 对上述两种策略的综合，定时检查并删除，通过限制删除操作执行时间和频率减少对CPU的占用时长

实现

定期删除使用activeExpireCycle完成，服务器周期性执行serverCron时会执行

```c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
	//...
    databasesCron(); /* Handle background operations on Redis databases. */
    //...
}
void databasesCron(void) {
	//...
    activeDefragCycle();
	//...
}
```

可以看到服务器的定时任务中会调用databasesCron()的定时任务处理数据库

```c
void activeExpireCycle(int type) {
    static unsigned int current_db = 0; // 当前检查进度，也就是当前检查到的数据库编号
    static int timelimit_exit = 0;
    static long long last_fast_cycle = 0; /* When last fast cycle ran. */
	//...
    for (j = 0; j < dbs_per_call && timelimit_exit == 0; j++) {
		//...
        redisDb *db = server.db+(current_db % server.dbnum);//对定位到当前应检查数据库
        current_db++;// 检查进度，++ 后移一个数据库索引
        //...
    }
}
```

函数每次运行对数据库进行检查，并抽取一定量随机键删除其中的过期键 ，当检测时间到的时候会停止检查，留待下次进行

难度

- 删除时间的设定
- 删除频率的时间

## AOF&RDB->过期键

### RDB

OUT

save或bgsave触发RDB，其中过期键并不会被存储到文件中

IN

当载入RDB文件时

- **主**服务器模式，过期键并不会被载入
- **从**服务器模式，**所有键**都会被载入

### AOF

OUT

键被惰性删除或者定期删除后会在AOF文件中追加一条DEL命令显示记录该键已被删除

对于一个已过期的键，使用get命令，执行流程

1. 删除数据库中的键
2. AOF文件追加DEL命令
3. 向调用者返回空恢复

AOF重写

过期键并不会被写入文件

复制模式

复制模式下，从服务器的删除操作由主服务器控制

- 主服务器删除过期键，显示向所有服务器发送DEL命令，告知服务器删除该键
- 从服务器处理过期键，并不会删除，而是以未过期键身份对待过期键
- 从服务器只有接到主服务器命令后会删除过期键
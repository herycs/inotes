# 事务

## 认识

事务通过对一整套命令进行打包，并且保证事务执行期间不会执行其他事务

multi, exec, watch

```shell
multi # 事务开始
      # cmd...
exec  # 事务结束
```

示例：

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name 123
QUEUED
127.0.0.1:6379> set age 12
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> get age
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "123"
4) "12"
127.0.0.1:6379>  
```

## 实现

实现流程

1. 事务开始
2. 命令入队
3. 事务执行

### 事务开始

multi命令执行

该命令执行将客户端从非事务状态切换到事务状态，通过修改REDIS_MULTI标识实现

服务器处理：

1. 接收到命令

2. 判定当前是否是事务状态

    - 否：-》立即执行

    - 是：-》命令是否是exec, discard, watch, mutli
        - 否：-》命令入队列，并返回QUEUED
        - 是：-》立即执行

### 命令入队

命令队列维护，multiCmd类型数组

multiCmd：参数 - 参数数量 - 命令指针

### 执行

exec

执行：

1. 创建空【回复队列】
2. 执行命令队列
3. 移除事务状态

## Watch

乐观锁，exec执行前监控任意数量的数据库键，若在事务执行前发生变动则拒绝执行事务

实例：

```shell
127.0.0.1:6379> watch name
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name 123
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> exec # 再执行这一步前在client2中执行 set name 333,导致name被修改
(nil)
127.0.0.1:6379>  
```

### 原理

每个数据库中都保存了watched_keys字典

redisDb结构中的dict *watched_keys

```shell
watched_keys |
key1         |--->client1--->client2
key2         |--->client3--->client4
```

当执行watch

```shell
watched_keys |
key1         |--->client1--->client2--->c99
key2         |--->client3--->client4--->c100
```

### 监视机制触发

当执行修改命令，调用touchWatchKey函数检查检测的键，若出现修改，则事务的安全性已被破坏，则将监视被修改键的客户端的REDIS_DIRTY_CAS标识打开

对于下述情况

```shell
watched_keys |
key1         |--->client1--->client2--->c99
key2         |--->client3--->client4--->c100
```

当key2被修改，则client3 & client4的标识都会被修改

## 安全性检查

执行事务前，判定客户端的REDIS_DIRTY_CAS标识是否被打开

- 是-》拒绝执行
- 否-》执行

## 事务的特征

事务不具有回滚特性，执行完毕哪怕出错，所有的命令都会被执行

之所以不设计事务是因为这和redis本身的追求不一致，作者认为事务的执行错误都是因为编程错误产生的是在开发环境下，而不是在生产环境下

## 可能错误

### 入队错误

命令不存在，格式不正确---》拒绝执行

### 执行错误

执行期间才可被发现，这些指令并不会修改数据库，也不影响事务一致性

### 服务器停机

可能的情况

- 服务器无持久化，重启之后数据库空白，数据一致性
- 服务器运行在RDB，数据库恢复后数据库还原到一致情况
- 服务器运行在AOF，AOF中有数据则重启后数据库恢复，否则数据库将是空白的
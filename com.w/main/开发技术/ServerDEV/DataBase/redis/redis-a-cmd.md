# Redis基础命令&配置

## 存取

### String

```shell
set key val
get key val
del key
```

批量操作

```shell
mset k1 v1 k2 v2
mget k1 k2
```

过期设置

```shell
127.0.0.1:6379> expire k1 5 --5秒过期
(integer) 1
127.0.0.1:6379> mget k1 k2
1) "v1"
2) "v2"
127.0.0.1:6379> mget k1 k2
1) (nil)
2) "v2"
```

setnx

SET if Not eXists

incr key

value为数值自增

getset k1 v1

设置值，并返回原值（没有则返回nil）

### List

lpush key value

rpush key value

lrange key start end

lindex key indexVal

```shell
127.0.0.1:6379> lpush i a b c
(integer) 3
127.0.0.1:6379> lindex i -1//倒数第一个
"a"
127.0.0.1:6379> lindex i -2
"b"
127.0.0.1:6379> lindex i -3
"c"
127.0.0.1:6379> lrange i 0 3
1) "c"
2) "b"
3) "a"
127.0.0.1:6379> lindex i 0
"c"
127.0.0.1:6379> lindex i 1
"b"
127.0.0.1:6379> lindex i 2
"a"
```

lpop key

rpop key

模拟队列

```shell
127.0.0.1:6379> rpush i1 0
(integer) 1
127.0.0.1:6379> rpush i1 2
(integer) 2
127.0.0.1:6379> rpush i1 3
(integer) 3
127.0.0.1:6379> rpush i1 4
(integer) 4
127.0.0.1:6379> lpop i1
"0"
127.0.0.1:6379> lpop i2
(nil)
127.0.0.1:6379> lpop i1
"2"
127.0.0.1:6379> lpop i1
"3"
127.0.0.1:6379> lpop i1
"4"
127.0.0.1:6379> lpop i1
(nil)
```

模拟栈

```shell
127.0.0.1:6379> rpush i2 a b c
(integer) 3
127.0.0.1:6379> rpop i2
"c"
127.0.0.1:6379> rpop i2
"b"
127.0.0.1:6379> rpop i2
"a"
127.0.0.1:6379> rpop i2
(nil)
```

### Hash

hset key field value

hget key field

hgetall key

hedl key field

### Set

sadd key value

smembers key

sismembers key value

值是否存在

scard key

获取长度

spop key

弹出

srem key value

### SortedSet

zadd key score value

zrange key start end [withscores]

以分数排序输出

```c
127.0.0.1:6379> zadd book 9 b1
(integer) 1
127.0.0.1:6379> zadd book 7 b2
(integer) 1
127.0.0.1:6379> zadd book 3 b3
(integer) 1
127.0.0.1:6379> zadd book 10 b4
(integer) 1
127.0.0.1:6379> zrange book 0 3	//正序
1) "b3"
2) "b2"
3) "b1"
4) "b4"
127.0.0.1:6379> zrevrange book 0 3 //逆序
1) "b4"
2) "b1"
3) "b2"
4) "b3"
```

zrangeby score1 score2

```c
127.0.0.1:6379> zrangebyscore book 3 8
1) "b3"
2) "b2"
```

zcrad key

获取长度

zrem key value

### 通用

keys *

type key

del key

extends key

### 复制

slaveof [ip] [port]

### 心跳

replconf ak [replication_offset]

## 哨兵

### 启动

redis-sentinel /path/xxx/xxx.conf

redis-server /path/xxx/sentinel.conf --sentinal

## 其他

### 估算最近一次命令请求执行数量

info status

### 对象引用值

object refcount key

### 对象空转时间

object idletime e

### 切换数据库

select numberOfDB

### 设置过期时间

setex key value

只针对字符串键，设置值同时设置过期时间

expire | expireat

设定键过期时间，单位：s

pexpire | pexpireat

设置键过期时间，单位ms

上述xxxat设置指定时间戳

上述设置过期时间命令最终都是pexpireat实现的

### 返回键剩余寿命

TTL | PTTL

```c
127.0.0.1:6379> expire w 123
(integer) 1
127.0.0.1:6379> ttl w
(integer) 118
127.0.0.1:6379> ttl w
(integer) 114
```

## 配置

### redis持久化机制

RDB：默认方式，不需要进行配置，默认就使用这种机制

在一定的间隔时间中，检测key的变化情况，然后持久化数据
	1. 编辑redis.windwos.conf文件
	
	after 900 sec (15 min) if at least 1 key changed save 900 1
	
	after 300 sec (5 min) if at least 10 keys changed save 300 10
	
	after 60 sec if at least 10000 keys changed save 60 10000

2. 重新启动redis服务器，并指定配置文件名称
     D:\JavaWeb2018\day23_redis\资料\redis\windows-64\redis-2.8.9>redis-server.exe redis.windows.conf	

AOF：日志记录的方式，可以记录每一条命令的操作。可以每一次命令操作后，持久化数据

1. 编辑redis.windwos.conf文件
	appendonly no（关闭aof） --> appendonly yes （开启aof）
	
	appendfsync always： 每一次操作都进行持久化
	
	appendfsync everysec： 每隔一秒进行一次持久化
	
	appendfsync no： 不进行持久化
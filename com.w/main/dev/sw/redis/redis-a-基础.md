# redis-a-基础

## 概述

### NoSQL类型数据库

> NoSQL(NoSQL = Not Only SQL)，意即“不仅仅是SQL”，是一项全新的数据库理念，泛指**非关系型**的数据库。
> ​		随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题。

#### 特点

**优点：**

成本：

- nosql数据库简单易部署，基本都是开源软件，不需要像使用oracle那样花费大量成本购买使用，相比关系型数据库价格便宜

查询速度：

- nosql数据库将数据存储于缓存之中，关系型数据库将数据存储在硬盘中，自然查询速度远不及nosql数据

存储数据的格式：

- nosql的存储格式是key,value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型

扩展性：

- 关系型数据库有类似join这样的多表查询机制的限制导致扩展很艰难

**缺点：**

​			1）维护的工具和资料有限，因为nosql是属于新的技术，不能和关系型数据库10几年的技术同日而语。
​			2）不提供对sql的支持，如果不支持sql这样的工业标准，将产生一定用户的学习和使用成本。
​			3）不提供关系型数据库对事务的处理。

#### 优势

性能高：

- 性能NOSQL是基于键值对的，可以想象成表中的主键和值的对应关系，而且不需要经过SQL层的解析，所以性能非常高。

可扩展性

- 同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平扩展。

### 关系型数据库的优势

支持复杂查询

- 可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。

能保证数据安全性

- 事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。

### **关系**

两类数据库互补，并非对立，一般会将数据存储在关系型数据库中，在nosql数据库中备份存储关系型数据库的数据

### 主流产品

**键值**(Key-Value)存储数据库
​			相关产品： Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
​			典型应用： 内容缓存，主要用于处理大量数据的高访问负载。 
​			数据模型： 一系列键值对
​			优势： 快速查询
​			劣势： 存储的数据缺少结构化
**列**存储数据库
​			相关产品：Cassandra, HBase, Riak
​			典型应用：分布式的文件系统
​			数据模型：以列簇式存储，将同一列数据存在一起
​			优势：查找速度快，可扩展性强，更容易进行分布式扩展
​			劣势：功能相对局限
**文档**型数据库
​			相关产品：CouchDB、MongoDB
​			典型应用：Web应用（与Key-Value类似，Value是结构化的）
​			数据模型： 一系列键值对
​			优势：数据结构要求不严格
​			劣势： 查询性能不高，而且缺乏统一的查询语法
**图形**(Graph)数据库
​			相关数据库：Neo4J、InfoGrid、Infinite Graph
​			典型应用：社交网络
​			数据模型：图结构
​			优势：利用图结构相关算法。
​			劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

## redis

### 概述

Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库，官方提供测试数据，50个并发执行100000个请求,读的速度是110000次/s,写的速度是81000次/s ，且Redis通过提供多种键值数据类型来适应不同场景下的存储需求。

### 应用场景

​			•	缓存（数据查询、短连接、新闻内容、商品内容等等）
​			•	聊天室的在线好友列表
​			•	任务队列。（秒杀、抢购、12306等等）
​			•	应用排行榜
​			•	网站访问统计
​			•	数据过期处理（可以精确到毫秒
​			•	分布式集群架构中的session分离

### 数据结构

 * redis存储的是key,value格式的数据

**key**

字符串

**value**

1. 字符串类型 string
2. 哈希类型 hash map格式 
3. 列表类型 list  linkedlist格式。支持重复元素
4. 集合类型 set 不允许重复元素
5. 有序集合类型 sortedset 不允许重复元素，且元素有顺序

### 管理

#### 基础

*通用命令*

```sql
keys *
--查看键的类型
type key
--删除指定键的value
del key
```

**String**

存

```sql
set key value
```

取

```sql
get key value
```

删

```sql
del key
```

**Hash**

存

```sql
hset key field value
--hset h1 name "zhangsan"
```

取

```sql
--取field的value
hget key field
--hget h1 name

--取所有field,value
hgetall key
--hgetall h1
```

删

```sql
hdel key field
--hdel h1 name
```

**list**

> 两种操作，在链表左侧操作，在链表右侧操作

存

```sql
lpush key value
rpush key value
```

取

```sql
lrange key start end
--lrange L1 0 3
```

删

```sql
lpop key
rpop key
```

**set**

存

```sql
sadd key value
```

取

```sql
smembers key
```

删

```sql
srem key value
```

**sortedset**

> 不允许重复元素，且元素有顺序.每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

存

```sql
zadd key score value
--score为评分，是后续排序依据
```

取

```sql
zrange key start end [withscores]
--可选项[withscores]，显示评分
```

删

```sql
zrem key value
```

### 持久化

> win

#### RDB

> **默认**方式，不需要进行配置，默认就使用这种机制

**定时**持久化：在一定的间隔时间中，检测key的变化情况，然后持久化数据

```shell
--编辑redis.windwos.conf文件
#   after 900 sec (15 min) if at least 1 key changed
save 900 1
#   after 300 sec (5 min) if at least 10 keys changed
save 300 10
#   after 60 sec if at least 10000 keys changed
save 60 10000
--重新启动redis服务器，并指定配置文件名称
D:\JavaWeb2018\day23_redis\资料\redis\windows-64\redis-2.8.9>redis-server.exe redis.windows.conf	
```

#### AOF

**日志**记录的方式，可以记录每一条命令的操作。可以每一次命令操作后，持久化数据

```shell
-- 编辑redis.windwos.conf文件
appendonly no（关闭aof） --> appendonly yes （开启aof）

# appendfsync always ： 每一次操作都进行持久化
appendfsync everysec ： 每隔一秒进行一次持久化
# appendfsync no	 ： 不进行持久化
```


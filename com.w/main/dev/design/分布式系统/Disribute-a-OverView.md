# 分布式-概述

## 集群

多节点完成相同任务

### 分类

**1.高可用集群**(High Availability Cluster)

高可用集群,普通两节点双机热备，多节点HA集群。

**2.负载均衡集群**(Load Balance Cluster)

常用的有 Nginx 把请求分发给后端的不同web服务器，还有就是数据库集群，负载均衡就是，为了保证服务器的高可用，高并发。

**3.科学计算集群**(High Performance Computing Cluster)

简称HPC集群。这类集群致力于提供单个计算机所不能提供的强大的计算能力。

### 作用

**负载均衡**：负载均衡能把任务比较均衡地分布到集群环境下的计算和网络资源。

**集群容错**：当我们的系统中用到集群环境,因为各种原因在集群调用失败时，集群容错起到关键性的作用。

## 分布式

业务拆分，分离部署

### 基本构成

数据：数据库集群

辅助：中间件（消息队列，搜索，文件，缓存）

业务：微服务集群

接口：接口集群

### 使用策略

垂直拆分

水平拆分

## 需要考虑的问题

### 运行

数据存储：数据一致性 | 时间一致性 | ID生成器 | 分布式Session

数据操作：分布式事务

并发：分布式锁 | 幂等性 | 大流量处理

### 开发

测试成本

### 运维

运维成本

## 挑战

### 异构的机器&网络

### 节点故障处理

### 不可靠的网络

## 评判

### 可扩展性

### 可用性与可靠性

### 高性能

### 一致性

## 相关技术

负载均衡：

- Nginx：高性能、高并发的web服务器；功能包括负载均衡、反向代理、静态内容缓存、访问控制；工作在应用层
- LVS： Linux virtual server，基于集群技术和Linux操作系统实现一个高性能、高可用的服务器；工作在网络层

webserver：

- Java：Tomcat，Apache，Jboss
- Python：gunicorn、uwsgi、twisted、webpy、tornado

service：SOA、微服务、spring boot，django

容器：docker，kubernetes

cache：memcache、redis等

协调中心：zookeeper、etcd等

- zookeeper使用了Paxos协议Paxos是强一致性，高可用的去中心化分布式。

rpc框架：grpc、dubbo、brpc

消息队列：kafka、rabbitMQ、rocketMQ、QSP

- 消息队列的应用场景：异步处理、应用解耦、流量削锋和消息通讯

实时数据平台：storm、akka

离线数据平台：hadoop、spark

dbproxy：cobar也是阿里开源的，在阿里系中使用也非常广泛，是关系型数据库的sharding + replica 代理

db：mysql、oracle、MongoDB、HBase

搜索：elasticsearch、solr

日志：rsyslog、elk、flume

> PS: apark、akka、kafka都是scala语言写的，看到这个语言还是很牛逼的

### 实现分布式主要的方式

**分布式应用用到的技术：** 网络通信，基于消息方式的系统间通信和基于远程调用的系统间通信。

**缺点：** 就是会增加技术的复杂度。

**基于Java自身技术实现消息方式的系统间通信：**
基于Java自身包实现消息方式的系统间通信的方式有：

- TCP/IP+BIO、TCP/IP+NIO
- UDP/IP+BIO、UDP/IP+NIO 

**TCP/IP+BIO** 在 Java 中可基于 Socket、ServerSocket 来实现 TCP/IP+BIO 的系统间通信。

Socket 主要用于实现建立连接及网络 IO 的操作，ServerSocket 主要用于实现服务器端端口的监听及 Socket 对象的获取。

多个客户端访问服务器端的情况下，会遇到两个问题:建立多个 socket 的，占用过多的本地资源，服务器端要承受巨大的来访量；创建过多的 socket，占用过多的资源，影响性能。

通常解决这种问题的办法是，使用 **连接池**，既能限制连接的数量，又能避免创建的过程，可以很大的提高性的问题。缺点就是竞争量大的时候造成激烈的竞争和等待。需要注意的是，要设置超时时间，如果不这样的话，会造成无限制的等待。

为了解决这个问题，采用一连接一线程的方式，同时也会带来副作用，内存占用过多。
TCP/IP 异步通信: JAVA NIO 通道技术实现。

### JAVA 分布式知识体系介绍

**第 2 章 大型分布式 Java应用与 SOA**
2.1 基于 SCA实现 SOA平台
2.2 基于 ESB实现 SOA平台
2.3 基于 Tuscany实现 SOA平台
2.4 基于 Mule 实现 SOA平台

**第 4 章 分布式应用与 SunJDK类库**
4.1 集合包
  4.1.1 ArrayList
  4.1.2 LinkedList
  4.1.3 Vector
  4.1.4 Stack
  4.1.5 HashSet
  4.1.6 TreeSet
  4.1.7 HashMap
  4.1.8 TreeMap
  4.1.9 性能测试
  4.1.10 小结
4.2 并发包（ java.util.concurrent ）
  4.2.1 ConcurrentHashMap
  4.2.2 CopyOnWriteArrayList
  4.2.3 CopyOnWriteArraySet
  4.2.4 ArrayBlockingQueue
  4.2.5 AtomicInteger
  4.2.6 ThreadPoolExecutor
  4.2.7 Executors
  4.2.8 FutureTask
  4.2.9 Semaphore
  4.2.10 CountDownLatch
  4.2.11 CyclicBarrier
  4.2.12 ReentrantLock
  4.2.13 Condition
  4.2.14 ReentrantReadWriteLock
4.3 序列化 /反序列化
  4.3.1 序列化
  4.3.2 反序列化

**第 5 章 性能调优**
5.1 寻找性能瓶颈
  5.1.1 CPU消耗分析
  5.1.2 文件 IO 消耗分析
  5.1.3 网络 IO 消耗分析
  5.1.4 内存消耗分析
  5.1.5 程序执行慢原因分析
5.2 调优
  5.2.1 JVM 调优
  5.2.2 程序调优
  5.2.3 对于资源消耗不多，但程序执行慢的情况

**第 6 章 构建高可用的系统**
6.1 避免系统中出现单点
  6.1.1 负载均衡技术
  6.1.2 热备
6.2 提高应用自身的可用性
  6.2.1 尽可能地避免故障
  6.2.2 及时发现故障
  6.2.3 及时处理故障
  6.2.4 访问量及数据量不断上涨的应对策略

**第 7 章 构建可伸缩的系统**
7.1 垂直伸缩
  7.1.1 支撑高访问量
  7.1.2 支撑大数据量
  7.1.3 提升计算能力
7.2 水平伸缩
  7.2.1 支撑高访问量
  7.2.2 支撑大数据量
  7.2.3 提升计算能力
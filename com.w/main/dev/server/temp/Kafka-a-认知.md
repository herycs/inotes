# Kafka

> 分布式流处理平台

## 理解

kafka可以理解为很长的日志文件，通过访问这个日志文件的offset的变化可以查阅从记录之初到现在的所有消息

offset由消费者控制，消息就可以消费多次

offset由kafka控制，(kafka设立单独Topic维护当前Topic的消费情况)消息就可以以指定的方式消费

## 应用

- 消息队列
- 流处理

## 基本概念

### 核心API

- The [Producer API](https://kafka.apachecn.org/documentation.html#producerapi) 允许一个应用程序发布一串流式的数据到一个或者多个Kafka topic。
- The [Consumer API](https://kafka.apachecn.org/documentation.html#consumerapi) 允许一个应用程序订阅一个或多个 topic ，并且对发布给他们的流式数据进行处理。
- The [Streams API](https://kafka.apachecn.org/documentation/streams) 允许一个应用程序作为一个*流处理器*，消费一个或者多个topic产生的输入流，然后生产一个输出流到一个或多个topic中去，在输入输出流中进行有效的转换。
- The [Connector API](https://kafka.apachecn.org/documentation.html#connect) 允许构建并运行可重用的生产者或者消费者，将Kafka topics连接到已存在的应用程序或者数据系统。

## 组成

### 概念

- broker：每个节点为一个Broker

- topic：发送到集群的消息以Topic区分， 每个topic至少有一个partition

- partition：partition物理上由多个segment组成，每一个segment存储着多个message信息（**默认是：1G**），而每一个message是由一个key-value和一个时间戳组成。

- Leader：每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition。

- Flower：

  Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。

  如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。

### 推拉

选择由 Producer 向 broker push 消息并由 Consumer 从 broker pull 消息。

### Topic分区

**轮询**：顺序分发，仅针对于**message没有key**的时候。

**Hash分区**：在message有key的情况下，（**key.hash%分区个数**）。如果在增加分区的时候，partition里面的message不会重新进行分配，随着数据的继续写入，这个新的分区才会参与load balance。

**自定义**：用户自己设定分区方式

### Offset

Kafka 会为每一个 Consumer Group 保留一些 metadata 信息——当前消费的消息的 position，也即 offset。

由Consumer 控制，所以 Kafka broker 是无状态的，它不需要标记哪些消息被哪些消费过，也不需要通过 broker 去保证同一个 Consumer Group 只有一个 Consumer 能消费某一条消息

## 读写

### 读取

Consumer会监听所有的partition,

若只有一个消费者，则监听所有分区，若有多个，则按规则分配（最多支持分区数量的消费者）

### 写入

leader负责写入

Producer 根据这个 key 和 Partition 机制来判断应该将这条消息发送到哪个 Parition

## 通信

### 一次通信过程

- Client向Server发送请求时
- Acceptor负责接收TCP请求，连接成功后传递给Processor线程；
- Processor线程接收到新的连接后，将其注册到自身的Selector中，并监听READ事件，根据TCP协议调用Handler进行处理
  - 二分查找定位index文件，log文件
  - 通过相对偏远量定位到具体的消息地址，读取消息
- 当Client在当前连接对象上写入数据时，会触发READ事件，
- Handler处理完成后，可能会有返回值给Client，并将Handler返回的结果绑定Response端进行发送

### 单播和广播

一个 Topic 可以对应多个 Consumer Group。

- 广播，只要每个 Consumer 有一个独立的 Group 就可以了。
- 单播，只要所有的 Consumer 在同一个 Group 里。

用 Consumer Group 还可以将 Consumer 进行自由的分组而不需要多次发送消息到不同的 Topic。

### Kafka delivery guarantee

- At most once 消息可能会丢，但绝不会重复传输
- At least one 消息绝不会丢，但可能会重复传输
- Exactly once 每条消息肯定会被传输一次且仅传输一次，很多时候这是用户所想要的。



**producer**

当 Producer 向 broker 发送消息时，一旦这条消息被 commit，因数 replication 的存在，它就不会丢。

如果 Producer 发送数据给 broker 后，遇到网络问题而造成通信中断，Producer 可以生成一种类似于主键的东西，发生故障时幂等性的重试多次，这样就做到了 Exactly once。

**Consumer**

读完消息commit，该操作会在 Zookeeper 中保存该 Consumer 在该 Partition 中读取的消息的 offset

读完消未 commit，下一次读取的开始位置会跟上一次 commit 之后的开始位置相同。

读完消息先 commit 再处理消息。这种模式下，如果 Consumer 在 commit 后还没来得及处理消息就 crash 了，下次重新开始工作后就无法读到刚刚已提交而未处理的消息，这就对应于 At most once

读完消息先处理再 commit。这种模式下，如果在处理完消息之后 commit 之前 Consumer crash 了，下次重新开始工作时还会处理刚刚未 commit 的消息，实际上该消息已经被处理过了。这就对应于 At least once。



如果一定要做到 Exactly once，就需要协调 offset 和实际操作的输出。精典的做法是引入两阶段提交。

high level API 而言，offset 是存于 Zookeeper 中的，无法存于 HDFS

low level API 的 offset 是由自己去维护的，可以将之存于 HDFS 中

## 日志

**segment**

组成： segment = index file 和 data file

**命名规则**：partion 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个 segment 文件最后一条消息的 offset 值。数值最大为 64 位 long 大小，19 位数字字符长度，没有数字用 0 填充。

```xml
000000...12223.index	000000...12223.log
1,23									message12201		23
2,345									message12202		345
...										...
N,position						message12223		position
```

### 日志格式

每个日志文件都是一个 log entrie 序列，每个 log entrie 包含一个 4 字节整型数值（值为 N+5）

"magic value"(1 个字节) +  CRC 校验码(4 个字节)+ 消息体(N 个字节)

 log entries = N *  segment

segment 以该 segment 第一条消息的 offset 命名并以“.kafka”为后缀。另外会有一个索引文件，它标明了每个 segment 下包含的 log entry 的 offset 范围

### 日志清理

- 基于时间
- 基于 Partition 文件大小

## 实践中问题

### 读写

消息队列的读请求非常高

可以将数据同步到第三方，例如：HDFS，通过转移读压力保证服务性能

### 扩容

以最新的partition作为数据进行写入，同步一段时间，确保所有消费者都跟上了消费

### mirror集群化

 Kafka 自带的 MirrorMaker 存在 2 点问题：

1. 被 Mirror 的 topic 是静态管理的，运维成本很高，且容易出错；

2. 一旦有 topic 增加或者减少，以及机器的加入或者退出，都会导致原有正在 Mirror 的数据断流，这主要是因为经历了停止服务，再启动服务的过程；

改进：

restController -> ZK {Topic1， Topic2}

restController

- 用于动态管理 topic、worker 节点的增减；
- 负责 TP 的分配策略，支持部分 partition 的迁移，这样新增节点或节点宕机会触发部分 TP 的迁移，不会造成 Mirror 服务的整体断流，仅仅是一小部分有抖动；
- 负责 worker 宕机的异常恢复；

worker

- 支持动态增加与减少 topic，这样增加或者减少 topic 避免了对已有 TP 传输的影响；
- 支持同时传输多个源集群到多个目标集群的数据传输能力；
- 支持将数据 dump 到 HDFS 中；

Zookeeper：

- 负责协调 controller 与 worker
- 有了restController管理 Kafka 多集群间的数据 Mirror，极大减少了我们的运维成本，以及出错的情况。此外，由于集群化管理的存在，我们可以快速地对 Mirror 服务进行扩缩容量，以便应业务突发的流量

### 资源隔离

隔离业务间标签，对Broker打标签，分配topic时确保服务的标签分配到不同机器上

### 队列阻塞

将消息以Topic进行hash拆分，当发现队列已满，则直接失败，避免对其他队列进行干扰，同时也确保了只阻塞一个队列

### cache污染

读写影响：Cache块是为了提高访问性能，但是若刚刚写入的数据就被刷新出内存块，这时读取会导致磁盘的刷新

follower影响：follwer同步过程会触发cache刷新，操作完成后内存cache 中会保留follwer使用过的数据，但是这些数据在之后是不会被读取的，这样会导致内存中驻留了大量的无效数据

处理：

生产者请求存入queue后请求就结束掉，之后启动多线程异步将数据写入磁盘

Follower请求照常进行

确保内存cache中都是生产者写入的数据
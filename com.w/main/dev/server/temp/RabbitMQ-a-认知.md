# MQ认知

## 评价

优点：

- 标注AMQP协议实现
- 高并发
- 集群部署简单
- 社区活跃度高

缺点：

- Erlang语言开发，导致进行二次开发难度较大

## 特性

- 支持多种消息传递协议，除了 AMQP 外，还可以通过插件支持所有版本的 STOMP 协议和 MQTT 3.1 协议
- 部署简单
- 支持跨语言开发，如：Java，.NET，PHP，Python，JavaScript，Ruby，Go
- 插拔式的身份验证和授权
- 可使用丰富的插件扩展

## 结构

Consumer <> Broker [  Exchange <> Binding <> Queue  ] <> Producer

- Queue { channelOne | channnelTwo | xxx 多个Channel }

- Virtual Host : 本质上就是一个 mini 版的 RabbitMQ 服务器

## 工作模式

### 消息分发模式

根据 Exchange 类型实现到Queue的消息分发模式

- direct - 单播
- fanout - 广播
- Topic - 主体模式

### 应用

1、Work queues

两个消费端共同消费同一个队列中的消息

2、Publish/Subscribe

每个消费者监听自己的队列

3、Routing

- 每个消费者监听自己的队列，并且设置routingkey。
- 生产者将消息发给交换机，由交换机根据routingkey来转发消息到指定的队列。

4、Topics

- 每个消费者监听自己的队列，并且设置带统配符的routingkey。
- 生产者将消息发给broker，由交换机根据routingkey来转发消息到指定的队列。

5、Header

header模式取消routingkey，使用header中的 key/value（键值对）匹配队列

6、RPC

## 通信过程

—–发送消息—–

1、生产者和Broker建立TCP连接。

2、生产者和Broker建立通道。

3、生产者通过通道消息发送给Broker，由Exchange将消息进行转发。

4、Exchange将消息转发到指定的Queue（队列）

—-接收消息—–

1、消费者和Broker建立TCP连接

2、消费者和Broker建立通道

3、消费者监听指定的Queue（队列）

4、当有消息到达Queue时Broker默认将消息推送给消费者。

5、消费者接收到消息。

## Queue

- 创建

生产者和消费者在推送消息时都会尝试创建对应的Queue

- 更新

对于已存在Queue，声明相同名称的Queue不会产生任何效果

### 死信队列

消息变为死信

- 消息被拒绝 (Basic.Reject/Basic.Nack) ，井且设置重回队列的参数 requeue 为 false；
- 消息过期
- 队列达到最大长度

channel.queueDeclare 方法中设置 x-dead-letter-exchange 参数来为正常队列添加死信交换器

## 消息

### 稳定性

- 提供了事务的功能。
- 通过将 channel 设置为 confirm（确认）模式。

### 避免丢失

- 消息持久化
- ACK确认机制
- 设置集群镜像模式
- 消息补偿机制

### 确认机制

正常情况下，消费者消费了消息后会返回ACK消息，消息之后会被删除

1.无ACK且链接中断，若是链接直接中断，则消息会被转发到下一个消费者

2.无ACK且链接未中断，若是链接未中断，则消息会被认为正常消费了，且消费者无能力继续消费下一个消息

### 拒绝

- 直接断开链接

  消息会被发送给其他消费者，坏处【增加了服务端开销】

-   basic.reject 命令

  RabbitMQ 2.0.0可以使用 basic.reject 命令，收到该命令RabbitMQ会重新投递到其它的Consumer

  如果设置requeue为false，RabbitMQ会直接将消息从queue中移除。

- 跳过无效消息

  当消息无效，直接ACK，不处理消息

### 持久化

**设置**

默认情况下，重启服务器会导致消息丢失掉

机制：持久化消息写入磁盘上的持久化日志文件，等消息被消费之后，Rabbit会把这条消息标识为等待垃圾回收。

1.Exchange持久化，在声明时指定durable => 1

2.Queue持久化，在声明时指定durable => 1

3.消息持久化，在投递时指定delivery_mode => 2（1是非持久化）

### 持久化成功需要满足的条件

下列四个条件均满足则，消息成功

- 声明队列必须设置持久化 durable 设置为 true.
- 消息推送投递模式必须设置持久化，deliveryMode 设置为 2（持久）。
- 消息已经到达持久化交换器。
- 消息已经到达持久化队列。

**消费情况**

- 持久化消息被消费

服务端收到ACK后会标记消息，之后会回收该消息

- 持久化消息未被消费

重启后会重建exchange和queue(以及bingding关系)， 通过持久化日志消息会被重建并放入对应的queues，exchanges

1. 投递消息的时候durable设置为true，消息持久化

   代码：channel.queueDeclare(x, true, false, false, null)，参数2设置为true持久化；

2. 设置投递模式deliveryMode设置为2（持久）

   代码：channel.basicPublish(x, x, MessageProperties.PERSISTENT_TEXT_PLAIN,x)，参数3设置为存储纯文本到磁盘；

**问题**

开启的持久化是针对磁盘操作的，故<u>性能</u>会变得低下

## 集群模式

### 默认模式

消息在集群中只存在一份

当请求打到其他节点时，消息会临时从当前节点被转发至被请求节点

例子：集群ABC，消息在A，这时请求B，消息会从A->传输到B，而后被消费

### 镜像模式

消息会自动在各个节点间同步

可以设置三种模式决定队列被镜像的方式

- All 队列会被同步到所有节点
- Exactly 队列同步到指定数量的节点上
- nodes 队列同步到指定节点

### 节点类型

1.内存节点：只保存状态到内存（一个例外的情况是：持久的queue的持久内容将被保存到disk）

2.磁盘节点：保存状态到内存和磁盘。

### 磁盘节点奔溃

唯一磁盘节点崩溃了，集群是可以保持运行的，但你不能更改任何东西。

### 节点停止流程

RabbitMQ 对集群的停止的顺序是有要求的，**应该先关闭内存节点，最后再关闭磁盘节点。**如果顺序恰好相反的话，可能会造成消息的丢失。
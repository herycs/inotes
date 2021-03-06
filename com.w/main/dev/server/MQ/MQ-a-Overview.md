# MQ

## 作用

### 优点

解耦：消息队列只在微服务中能起到解耦作用)。

异步：一个简单的异步锁。

削峰：削弱高峰时候的冲击。

### 缺点

系统可用性降低：那消息队列挂了？？？

系统复杂性增加：要多考虑很多方面的问题，比如一致性问题、如何保证消息不被重复消费，如何保证保证消息可靠传输。

## 比对

**ActiveMQ：**

java语言开发的

单机吞吐量：万级

时效性：ms级

可用性：高(主从架构)

功能特性：成熟的产品，在很多公司得到应用；有较多的文档；各种协议支持较好

**RabbitMQ：**

erlang语言开发的，

单机吞吐量：万级，

时效性：us级，

可用性：高(主从架构)，

功能特性：基于erlang开发，所以并发能力很强，性能极其好，延时很低;管理界面较丰富

**RocketMQ：**

java语言开发的，单机吞吐量：10万级，时效性：ms级，可用性：非常高(分布式架构)，

功能特性：MQ功能比较完备，扩展性佳

**kafka：**

scala语言开发的，单机吞吐量：10万级，时效性：ms级以内，可用性：非常高(分布式架构)，

功能特性：只支持主要的MQ功能，像一些消息查询，消息回溯等功能没有提供，毕竟是为大数据准备的，在大数据领域应用广。

## 选取

(1)中小型软件公司，建议选RabbitMQ.一方面，

- erlang语言天生具备高并发的特性，而且他的管理界面用起来十分方便。
- 定制化开发erlang的程序员不多呢？所幸，RabbitMQ的社区十分活跃，可以解决开发过程中遇到的bug，这点对于中小型公司来说十分重要。
- 数据量：不考虑rocketmq和kafka的原因是，一方面中小型软件公司不如互联网公司，数据量没那么大，选消息中间件，应首选功能比较完备的，所以kafka排除。不考虑rocketmq的原因是，rocketmq是阿里出品，如果阿里放弃维护rocketmq，中小型公司一般抽不出人来进行rocketmq的定制化开发，因此不推荐。

(2)大型软件公司，根据具体使用在rocketMq和kafka之间二选一。

- 一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。针对rocketMQ,大型软件公司也可以抽出人手对rocketMQ进行定制化开发，毕竟国内有能力改JAVA源码的人，还是相当多的。
- 至于kafka，根据业务场景选择，如果有日志采集功能，肯定是首选kafka了。
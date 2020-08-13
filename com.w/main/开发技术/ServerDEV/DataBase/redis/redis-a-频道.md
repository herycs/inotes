# 频道

## 订阅与退订

订阅者会接收到向频道发送的信息，执行推送消息的客户端会得到一个返回值（消息被接受的次数）

## 底层维护

### 订阅&退订

频道：

订阅关系维护在客户端中的pubsub_channels字典内，k - 频道，v - 链表（记录所有订阅的客户端）

```
pubsub_channels
channel-1       |---->client1---->client2
channel-2       |---->client2---->client4
```

订阅&退订分别对应着在链表中节点的增加&删除

模式：

模式订阅和退订维护在pubsub_pattern属性里面

```
pubsub_pattern---> | pubsubPattern-1 ---> | pubsubPattern-2 |
				  | client-1			 client-3
				  | client-2			 client-4
				  | "pattern-1"		     "pattern-2"
```

结构：

订阅退订同理

### 发送

1. 发送给channel
2. 发送给所有匹配的模式订阅者

### 查询订阅情况

频道：

pubsub numsub [channel-name] [channel2-name]

模式：

pubsub numpat [name1] [name2]
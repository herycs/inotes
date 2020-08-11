# 集群

## 基础

集群通过分片进行数据共享，提供复制和故障转移的功能

连接节点：cluster meet [ip] [port]

启动节点：集群模式下的redis服务器，cluster-enable = yes

集群状态下的节点任然会使用单机模式中的服务器主键，除此之外，使用redisServer保存服务器状态，redisClient保存客户端状态，集群模式下的数据结构存储到clusterNode结构、clusterLink结构以及clusterState

## 集群数据结构

clusterNode 保存集群节点的基础信息

clusterLink 保存连接节点有关信息

二者相似点：都有套接字描述符、输入、输出缓冲区

二者区别：clusterNode套接字用于连接客户端，clusterLink用于连接节点

## 命令实现

cluster meet

1. A为B建立clusterNode结构，并添加到clusterState.nodes字典中
2. A--->B：meet
3. --->B：meet，B为A建立clusterNode结构，并添加到clusterState.nodes字典中
4. A<---B：PONG
5. A--->B：PING
6. A<---B：PONG，握手完成

之后A将节点B信息通过Gossip协议传播给集群中其他节点

## 槽指派

将槽指派给节点：cluster addslots [firstID secondID ... lastID]

记录节点的槽指派信息，clusterNode 中的 solts 是一个二进制数组，16384 / 8 = 2048Byte，共16384个二进制位

索引：0 1 2 3 ... 16384，对应位置的 1 表示节点负责此槽点

### 槽指派信息传播

节点会将solts信息发送到其他节点

每个节点的solts数组保存自己保存的节点信息，同时会以接收到的信息更新clusterState.nodes字典中查找节点B对对应的clusterNode结构信息

solts数组中每个位置会有个指针指向被指派到的clusterNode结构中

### 客户端指派

客户端发送操作数据库键的指令

- 键指派的槽正好指派给了当前这个节点，那么这个节点直接执行指令
- 键不在当前槽，节点会返回给客户端MOVED指令，将客户端引导至正确节点(反馈正确槽和节点的ip port)

判定键是否在当前槽：CRC16(key) & 16383，计算key的校验和

cluster keyslot "key name"

### MOVED 格式

moved [solt] [ip] : [port]

客户端处理是直接转换，而不会在客户端打印错误，只是提示转移信息

### 数据库

节点只可使用 0 数据库

除了k-v保存到数据库中，clusterState节点中的slots_to_keys跳跃表保存槽和键的关系

## 重分片

具体操作由Redis集群管理软件redis-trib负责执行

流程：

1. 向目标发送setslot，让源节点准备slot键值对迁移至目标节点 cluster setslot [slot] importing [source_id]
2. 向源发送，cluster setslot [slot] migrating [target_id]
3. 向源节点发送，cluster getkeysinslot [slot] [count]
4. 对于3中获得的k，向源发送migrate [target_id] [target_port] [key_name] 0 [timeout]
5. 重复3,4，直至完成
6. 发送给任意一个节点，cluster setslot [slot] node [target_id]，将槽分给目标结点，通过信息传递给整个集群

### ASK错误

当在迁移期间接收到命令，则选择处理策略

- 若key在当前节点中，则处理
- 否则返回ask错误
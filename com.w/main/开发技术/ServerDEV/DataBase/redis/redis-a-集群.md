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
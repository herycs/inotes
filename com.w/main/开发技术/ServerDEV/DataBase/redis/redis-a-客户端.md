# 客户端

redis本身就是1->n服务程序结构

## 基础

### 结构

```c
struct redisServer {
	list *clients;
}
```

### 属性

> - 通用属性
> - 基本属性

#### fd

redisClient中的int fd属性描述了客户端正在使用的套接字描述符

伪客户端：fd = -1，命令请求来源基于AOF或者Lua脚本，使用到伪客户端的情况：1.载入AOF文件还原数据库，2.使用Lua脚本执行redis命令

普通客户端：fd > -1，使用套接字与服务端交互

列出所有客户端

```shell
127.0.0.1:6379> client list
addr=127.0.0.1:14887 fd=6 name= age=90 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

为客户端设置名字

client sername

#### flags

int flags

标志了客户端的角色

- 主从复制时：主服务器时从服务器的客户端，从服务器是主服务器的服务端，REDIS_MASTER，REDIS_SLAVE
- 低版本：REDIS_PRE_PSYNC，版本低于2.8主服务器不能使用PSYNC命令同步，这个标志只能是REDIS_SLAVE处于打开状态时使用
- REDIS_LUA_CLIENT标识客户端是专门用于处理Lua脚本里面Redis命令的伪客户端

除此之外还有一些其他的标志状态

#### querybuf

sds querybuf

保存客户端发送的命令请求

大小依据内容动态变化，最大不可超过1G，否则该客户端将被关闭

命令与参数：服务对客户端的命令请求保存到	客户端状态的 querybuf属性之后，对命令内容进行解析（保存：命令参数-》argv，参数个数-》argc）

#### 输出缓冲区

每个客户端有两个输出缓冲区

1. 大小可变：保存恢复长度比较大的内容
    - buf：字节数组
    - bufpos：字节数组数量
2. 大小不可变：回复较短内容，例如OK

#### authenticated

记录客户端是否通过了身份验证，0-》未通过，1-》通过

#### ctime

客户端创建时间，用于计算和Server连接了多久

#### lastinteraction

最后一次和server互动

## 创建&关闭

### 创建

客户端通过调用connect函数连接服务器，调用对应的事件处理器，设置对于的客户端状态

### 关闭

- 进程退出或者被杀死，网络连接关闭-》客户端关闭
- 客户端发送不合规范的数据，服务端-》杀死
- 客户端成为了client kill 目标，关闭
- 客户端配置了time out时间限制，空转超时-》关闭
- 输入缓冲区数据过大，超过1G-》关闭
- 要恢复的结果超过输出缓冲区大小，客户端-》关闭
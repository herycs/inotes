# Redis哨兵-Sentinel

## 作用

高可用解决方案：一个或者多个sentinel监视任意多个主服务器，在主服务器下线时将其从服务器升级

## 过程

1. 主服务器Master1下线
2. 设置Master1的下属服务器slave1为主服务器Master，其余Slave2，Slave3不变
3. 向所有从服务器发送复制指令，使得这些Slave成为新的Master的从服务器
4. Master1上线，设置其为Master1的从服务器

## 初始化

redis-sentinel /path/xxx/xxx.conf

redis-server /path/xxx/sentinel.conf --sentinal

执行任务：

1. 初始化服务器
2. 普通服务器代码替换为Sentinel代码
3. 初始化Sentinel状态
4. 依据配置文件初始化监视清单
5. 创建向主服务器的网络连接

### 初始化服务器

本质，特殊模式下的Redis服务器

### 使用专用代码

端口号：6379->26379

命令列表：旧替换为新的，Sentinel只支持7个命令

### 初始化Sentinel状态

就是初始化Sentinel结构

### 初始化信息

初始化Sentinel的Masters字典

### 创建网络连接

一共两个连接：命令连接 | 订阅连接

订阅连接确保消息接收不会遗失

命令连接提供向主服务器发送的命令的支持

### 获取主服务器信息

info获取两方面信息：Master信息 | Master下所有Salve信息

服务器name = ip : port

### 更新Sentinels字典

保存除Sentinel本身外，所有同样监视这个主服务器的其他Sentinel资料

### 创建Sentinel连接

当发现新的Sentinel则会增加向新的Sentinel的命令连接

### 主观下线检测

每1s向所有关联实例发送Ping命令，包括从服务器，主服务器，其他Sentinel

### 客观下线检测

向其他Sentinel询问下线状态，若同样式下线状态，则进行故障转移

## Sentinel选举

> 每次选举之后，所有服务器纪元+1
>
> 领头Sentinel设定后，一个纪元内不可更改
>
> 发现主服务器下线的Sentinel要求其他Sentinel设置自己为局部Leader
>
> 局部Leader设置原则，先到先得

1. 当主服务器下线，选举主Sentinel对下线主服务器进行故障转移
2. 源Sentinel发送给其他Sentinel命令：sentinel-is-master-down-by-addr [ip]，若ip是源Sentinel的runID，则表明要求将源Sentinel设置为后者的局部Leader
3. 接收到Sentinel命令的Sentinel回复信息：其中leader_runid 代表Sentinel的局部LeaderID，leader_epoch为配置纪元
4. Sentinel被半数Sentinel设置为局部Leader则就会成为全局Leader

给定时间未得出结果，则下次进行，直至出现Leader

## 故障转移

1. 下线主服务器的从服务器中提拔一个作为新Master
    1. 删除离线或者断线状态服务器
    2. 删除列表中最近5秒内未通信过的从服务器
    3. 删除连接超时服务器
    4. 分别依据：优先级 | 偏移量 | 服务器ID 为参考因素选择服务器，ID选最小的
2. 通知所有下线服务器从服务器复制新的Master
3. 已下线服务器设置为新主服务器的从服务器，下线服务器上线后成为新主服务器的从服务器

### 选取新主服务器

选择状态良好，数据完整从服务器作为主服务器，向服务器发送slaveof no noe
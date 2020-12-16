# 稳定性

## 在线日志分析

异常堆栈信息，快照

### 日志分析命令

查看文件内容

显示内容：cat

分页显示：more

更加丰富：less，支持查找，支持高亮显示

文件尾：tail

文件头：head

内容排序：sort

字符统计：wc

重复出现的行：uniq

字符串查找：grep

文件查找：find

表达式求值：expr

归档文件：tar

url访问：curl

查看请求访问量：cat access.log | cut -fl -d " " | sort | uniq -c | sort -k 1 -n -r | head -10

查看最耗时的页面：cat access.log | sort -k 2 -n -r | head -10

统计404请求的占比：cat ...

## 日志分析脚本

### sed编辑器

不修改文件本身

### awk程序

### shell脚本

## 集群监控

### 监控指标

load

运行队列中的平均线程数

CPU利用率

网络traffic：sar -n DEV 1 1 查看网络状况

磁盘IO：iostat -d -k

内存使用：free -m、vmstat

qps(query per second)

rt(response time)

select/ps

update/ps,delete/ps

GC

## 心跳检测

ping

(潜艇人员军事术语，表示发射声波脉冲)

应用层检测

业务检测

## 容量评估及应用水位

当前水位 = 当前总qps / (单台机器极限qps * 机器数) * 100%

## 流量控制

### 流量控制实施

多余流量直接放弃

- 例如利用Java信号量

单机内存队列进行有限等待

分布式消息队列来将用户请求异步化

- 后端系统固定频率消费请求

## 服务稳定性

### 依赖管理

### 优雅降级

通过依赖管理，通过当前系统依赖服务以及运行主流程判定服务是否会影响应用的主流程，来决定服务优先级

### 服务分级

必要情况下丢弃低级服务请求

### 开关

必要情况下直接关闭非核心服务

### 应急预案

应急预案，故障演练

## 高并发系统设计

### 操作原子性

### 多线程同步

synchroized

### 数据一致性

### 系统可扩展性

### 并发减库存

## 性能优化

### 寻找瓶颈

- 前端
- 服务端
- 操作系统
- 数据库查询
- JVM调优

前端优化

YSlow

页面响应时间

方法响应时间

GC日志分析

数据库查询，慢日志

系统资源使用

### 性能检测工具

ab：apche基金会提供的用于对Http服务器进行性能测试的工具

Jmeter

VisualVM

HP LoadRunner

反向代理引流

TCP Copy：复制线上请求，可在不上线情况下进行实际情况测试

### 性能优化

前端优化

- 页面HTTP请求数量
- 是否使用CDN网络
- 是否压缩

Java程序优化

- 单例
- Future模式
- 线程池
- 选择就绪（NIO机制）
- 减少上下文切换

压缩

结果缓存

数据库查询优化性能

- 合理使用索引
- 反范式设计
- 使用查询缓存
- 使用搜索引擎
- 使用key-value数据库

GC优化

硬件

## 故障排查

jps

jstat

jinfo

jstack

jmap

BTrace：Java程序动态追踪

Memory Analyzer（MAT）

VisualVM


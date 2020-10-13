# Hadoop概述

## 背景

### 需要解决的难点

- 读取速度：多磁盘并行读取
- 硬件故障：备份，冗余磁盘阵列
- 任务需求数据量大：分析需要多个盘的数据结合使用

### 特征

- 数据本地化
- 无共享

Hadoop提供了可靠的存储分析系统，核心两部分HDFS（Hadoop Distributed FileSystem)实现存储，MapReduce实现分析处理

> MapReduce 将磁盘读写问题进行抽象，并转换为数据集，计算由map和reduce两部分组成
>
> 每次查询需要处理整个数据集

### B树与MapReduce

少量的数据更新B树性能更好（受限于寻址的比例）

更新大部分数据时MapReduce更合适，适用于少更新，多读取的数据操作

### 网格计算

高性能计算（High Performance Computing, HPC）和网格计算（Grid Computing）组织致力于研究大数据计算

让程序员仅从键-值角度考虑任务的执行

MapReduce负责处理部分数据失效，进程之间的协调

MapReduce负责控制 mapper 的输出结果传递给reducer的过程

### 设计目标

服务于短时间可以执行完成的任务，运行于内部通过高速网格连接的单一数据中心内


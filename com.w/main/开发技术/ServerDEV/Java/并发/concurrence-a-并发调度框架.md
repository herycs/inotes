# 调度框架

# Executor框架

### 两级调度模型

HotSpotVM线程模型中Java线程被一对一映射为本地OS线程

# Fork/Join

### 工作窃取算法

- 优点：充分利用线程进行并行计算
- 缺点：某些情况下仍然存在竞争

### 框架设计

1. 分割任务
2. 执行任务并合并结果

### 异常处理

- 查看是否有异常：isCompletedAbnormally()
- 获取异常：ForkJoinTask.getException()

### 实现原理

- ForkJoinTask[]
- ForkJoinWorkerThread[]

任务状态四种：完成（NORMAL），取消（CANCELLED），信号（SIGNAL），异常（EXCEPTIONAL）
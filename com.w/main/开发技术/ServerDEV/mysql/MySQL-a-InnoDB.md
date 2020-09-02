# InnoDB

> 多线程存储引擎

## 后台线程

### Master Thread

职责：

- 负责缓存池数据到磁盘的异步刷新
- 负责数据的一致性保证，脏页刷新，合并插入缓冲等

1.0.x之前

- 最高优先级
- 内部多个Loop构成（主loop，后台backgroup loop，刷新flush loop，暂停suspend loop），主线程依据运行状态在各个循环间切换

### IO Thread（AIO处理）

- 1.0之前，4个IO，write, read, insert buffer, log IO thread
- 1.0.x开始，write * 4, read * 4, insert buffer, log IOthread

### Purge Thread

- 回收已经被使用的undo页
- 1.1之前Master线程中处理
- 1.1之后可单独起一个线程处理

### Page Cleaner Thread

- 1.2.x引入
- 负责：脏页的刷新操作

## 内存

### 缓冲池

- 单位：页(默认16kb)
- 数据读取，现在缓冲池中查看是否有，是则（缓存命中）
- 数据修改，先改缓冲，再刷新到磁盘
- 1.0.x开始允许有多个缓存池实例，将页以hash值为依据分配到不同的池中

参数：innodb_buffer_pool_instance

### LRU List, Free LIst, Flush List

采用LRU 最近最少使用算法（优化过，LRU List中增加，midpoint位置）

- 新增加的页存放于midpoint位置（默认是：5/8处）
- Innodb中称：5/8前数据为new，后3/8为old
- 1.0.x之后支持页压缩（压缩后页小于16KB，由unzip_LRU 管理）

参数：

- midpoint位置：innodb_old_blocks_pct
- midpoint位置数据多久被增加到：热数据表中

申请内存过程

1. （需求4kb)，检查4kb unzip_LRU 列表是否有空闲页，有直接使用
2. 检查是否有，8kb unzip_LRU列表是否有空闲页，有，页分成两个4kb，增加到4kb unzip_LRU列表中
3. LRU列表申请16KB，分为8KB + 2 * 4KB，

脏页

存在于：LRU列表 + Flush列表，LRU管理页的可用性，Flush负责页的刷新

### Redo log buffer

redo日志，先存于buffer，再刷新到磁盘（一般间隔1s）

触发刷新到磁盘情况：

- Master Thread每秒将重做日志刷新到磁盘
- 事务提交时将日志刷新到磁盘
- 缓冲空间空闲小于1/2

### 额外内存池

innodb内存管理采用的内存堆的方式，申请一些数据结构本身进行分配时从额外内存中申请，申请不成功才从缓冲池中申请

### CheckPoint

日志过大，从头恢复时间很长，检查点可有效缩短时间

## 关键特性

- 插入缓冲
- 两次写
- 自适应hash索引
- 异步IO
- 刷新邻接页

### Insert Buffer

使用条件

- 索引是辅助索引
- 索引不唯一

流程

- 插入的聚簇索引页在缓冲池中，则直接插入
- 插入的聚簇索引页不在缓冲池中，则先存到 insert buffer ,以一定的频率进行insert buffer 和 辅助索引页子节点 的merge，完成多个插入的合并操作

### Change Buffer

1.0.x引入

增加：Delete Buffer， Purge Buffer

### Merge Insert Buffer

insert buffer中的记录何时合并到真正的辅助索引？

1. 辅助索引页被读取到缓冲池
2. Insert Buffer Bitmap 页（用于追踪辅助索引页的可用空间）追踪到该辅助索引页时已无可用空间（可用空间少于1/32页）
3. Master Thread

### 两次写

Insert Buffer + Double Write

### 自适应Hash索引

adaptive hash index，AHI，通过缓冲池的B+树页构造而来，依据访问频率和访问模式建立hash索引

访问模式就是查询条件，若查询条件不一样，例如第一次：a == b, 第二次a == b and c == b，则不会构建索引

### 异步IO

InnoDB 1.1.x之前是代码中的实现，自1.1.x开始提供了内核级别的实现（需要有libaio库的支持，安装MySQL时提示libaio.so.1就是因为这个库没有）

### 刷新邻接页

> 可开关

对于传统机械硬盘而言，可以提升一定的性能，但对于IOPS高的磁盘而言，建议关闭

另外一个问题就是有没有必要使用，刷新邻接页的目的是为了提高性能，若开始此特性导致性能降低，则没必要使用

## 启动&关闭&恢复

### 关闭

innodb_fast_shutdown={0, 1(default), 2}

0---关闭时完成，所有full purge, merge insert buffer并将所有脏页刷新回磁盘

1---缓冲池中数据脏页刷新回磁盘

2---日志写入日志文件，留待启动后恢复

### 恢复

innodb_force_recovery = [1,6]

1---忽略检查到的corrupt页

2---阻止master thread线程运行

3---不进行事务回滚

4---不进行插入缓冲合并

5---不查看撤销日志（Undo log)

6---不进行前滚操作

设置值[>0]后，用户可对表执行select, create, drop, 但不允许 insert, update, delete这类DML操作不允许
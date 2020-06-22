# 存储引擎

# 基础概念

- 数据库：文件的集合（物理磁盘文件+其他类型文件）
- 实例：运行于内存中的（线程+线程共享内存区域）

关系：1 to 1，集群模式下例外

MySQL单进程多线程

## 存储引擎概览

### InnoDB

面向：在线事务处理应用OLTP

5.8.8之后为默认存储引擎

特点：行锁设计，支持外键

高并发性基础：MVCC

隔离级别：REPEATABLE，采用next-key locking避免幻读

其他：插入缓冲，二次写，自适应hash索引，预读

索引类型：聚簇，以ID主键作为基础顺序排序，若不存在指定主键，则为每一行生成一个ROWID作为主键

### MyISAM

面向：OLAP应用

特点：不支持事务，表锁，支持全文索引，缓冲池只缓冲索引文件

5.8.8之前默认存储引擎

构成：MYD（数据文件） + MYI（索引文件）

### NDB

面向：集群存储引擎

特点：数据全在内存中（5.1之后非索引数据可存放在磁盘）

待优化：JOIN操作在数据库层面完成，不是在存储引擎层面完成

### Memory

面向：临时数据，临时表

特点：数据存储在内存中，变长字段使用定长存储，不支持TEXT，BLOB类型

锁：只支持表锁

索引：采用hash索引

使用：MySQL使用此存储引擎作为查询中间结果集，当列类型有TEXT，BLOB时会转换到MyISAM存储引擎中存储到磁盘

### Archive

面向：高速插入，压缩功能

特点：只支持INSERT和SELECT操作，5.1开始支持索引，数据行压缩存储（zlib算法，压缩率10:1）

锁：行锁，非事务安全

### Federated

面向：远程数据库引用

特点：本身不存放数据，指向远程的数据库服务器的表

### Maria

面向：MyISAM的取代版本

特点：支持数据、索引文件缓存

锁：行锁

事务：提供MVCC，支持事务和非事务的安全选项

提供更好BLOB字符类型的处理性能

查看命令：show engines

## MySQL 连接

- tcp/ip
- 管道命名、共享内存
- UNIX域套接字

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

### 插入缓冲

使用条件

- 索引是辅助索引
- 索引不唯一

流程

- 插入的聚簇索引页在缓冲池中，则直接插入
- 插入的聚簇索引页不在缓冲池中，则先存到 insert buffer ,以一定的频率进行insert buffer 和 辅助索引页子节点 的merge，完成多个插入的合并操作

### Change Buffer

1.0.x引入

增加：Delete Buffer， Purge Buffer


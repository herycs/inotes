# 存储引擎

> 存储引擎基于表，而不是基于数据库

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

# 文件

> - 参数文件
> - 日志文件
> - socket文件
> - pid文件
> - MySQL表结构文件
> - 存储引擎文件

## 参数文件

静态&动态变量

## 日志文件

1. 错误日志（error log)

2. 二进制日志（binlog）

    对数据库执行更改的所有操作

    作用：恢复，复制和审计

3. 慢查询日志（slow query log）

    long_query_time = 10(default，单位：s)

    运行时间[>long_query_time]会被记录下

4. 查询日志（log）

    主机名.log

## Socket文件

UNIX系统下使用本地连接可以使用套接字文件

## pid文件

主要提供给系统使用

## 表结构定义文件

跨存储引擎*.frm文件

## 存储引擎文件

### InnoDB存储引擎文件

#### 表空间文件

存储依照表空间存储，默认文件ibdata1(10M)

#### 重做日志文件

每个InnoDB存储引擎至少一个重做日志组，组成ib_logfile0 + ib_logfile1，两个文件循环写入

# 表

## 索引组织表

创建索引，信息来源（1.主键，2.Unieuq not null非空唯一主键，3.自动创建6字节大小指针）

## 逻辑存储结构

构成row--->page--->extent（区)--->段 + 其他段--->表空间

### 表空间

包含所有数据，由多个段构成

### 段

分类有数据段，索引段，回滚段等

### 区

每个区1M，由连续的页构成，为保证连续性，一次会申请4~5个区，默认情况下页大小为16kb(64个连续的页)

InnoDB 1.0.x引入压缩页，1.2.x新增innodb_page_size

每个段开始使用32个连续的页存放数据，使用完后是64个连续的页的申请

### 页

### 行

## 行记录格式

### Redundant

兼容以前版本

格式：变长字段列表（逆序存放）| 记录头信息 | 列数据（多项）

char类型NULL值占用空间

### Compact

5.1开始

格式：变长字段列表（逆序存放） | NULL标志位 | 记录头信息 | 列数据（多项）

**隐藏列**：事务ID列（6字节），回滚指针列（7字节）

若没有定义主键还会有rowid列（6字节）

对于变长字段未使用空间使用0x2c填充

NULL值不占用空间

### 行溢出数据

以BLOB为例，数据页保留部分数据（前768字节），大量数据存储在BLOB页中

### Compressed&Dynamic

旧文件格式：Antelope：Compact， Redundant

新文件格式：Barracuda：Compressed，Dynamic

新的文件格式中BLOB中的数据采用**完全**行溢出，数据页只存储20字节指针，实际数据都在Off Page中

### Char行存储结构

4.1开始，char(N)中N表示为字符长度，而不是字节长度

对于多字节字符编码的Char数据类型存储，InnoDB内部将其视为边长字符类型

## 数据页结构

- File Header(38Byte)
- Page Header(56Byte)
- Infimun&Supremum Records
- Users Records
- Page Directory
- File Trailer(8byte)

Infimun&Supremum Records用于限定记录的边界Infimun比页中所有可能值都小，Super同理

Page Directory存放记录的页的相对位置

## 约束

### 数据完整性

- 主机
- 外键
- 唯一
- 非空
- 默认

### 错误数据约束

某些情况下支持错误数据的写入操作，例如：非空限制列，写入NULL字段

实际操作时会在内部进行转换，再写入

若要严格控制可以设置其检查级别

### ENUM&SET

可以限制数据内容

### 触发器

可以设置一些操作的处理，例如：异常操作写入日志等

### 外键约束

父子表之间的操作，依据不同情况，对父子表数据进行更新以及更新的检查

### 视图

### 物化视图

## 分区表

> 不是在存储引擎层完成，但不是所有存储引擎都支持

分区，将一个表或索引分解为更小更加利于管理

### 类别

- RANGE分区
- LIST分区
- HASH分区
- KEY分区
- COLUMNS分区
- 子分区

NULL值

视NULL小于任何一个非NULL

### 性能

访问模式需要研究，否则可能对性能产生反效果

### 表和分区数据交换

# 索引与算法

InnoDB支持索引：B+，Hash，全文索引

## 分类

InnoDB中分为聚簇索引和辅助索引，内部都是B+树，区别是叶子节点存储到是否是一整行

### 聚簇索引

实际表依照B+树进行排序，每张表只能拥有一个聚集索引

### 辅助索引

叶子节点不包含全部行数据，除了包含键值外还有一个书签，书签告诉存储引擎在哪可以查到数据（由于InnoDB是索引组织表，故书签就是相应的聚簇索引键）

查数据流程：遍历辅助索引查找目标值（得到聚簇索引键），查找聚簇索引（得到实际数据）

## Cardinality

当一个列的数据具有高度选择性时，就适合建立索引

## 使用

### 联合索引

一个字段定位不到，需要多个字段辅助定位，B+树结构中的节点键值数量[>=2]

### 覆盖索引

从辅助索引中就可以查询到记录，且这种索引远小于聚簇索引

### 优化器的选择

访问的数据占整个表很大一部分时（20%左右）会使用聚簇索引，而放弃其他索引

### 索引提示

Index Hint可以显示告知优化器使用的索引

使用场景：1.优化器选择错误；2.优化器可选索引很多，导致在选择上花费较多时间

### Multi-Range Read优化

5.6开始支持，目的使用此优化减少磁盘的IO，随机改为顺序

### Index Condition Pushdown优化

5.6开始支持，在取出数据同时进行where条件过滤（将过滤在存储引擎层执行了，减少了上层SQL层对数据的索取）

## Hash索引

除法散列，缓冲池中的Page页使用一个Chain指针指向相同Hash值的页

InnoDB存储引擎对于其中的页如何进行查找的呢？

表空间有个space_id，用户查找的表空间为某个16KB的页offset

存储引擎将space_id左移20位 + space_id + offset，从而定位到槽中

### 自适应Hash

存储引擎自己控制，默认为开

## 全文索引

InnoDB 1.2.x开始，存储引擎支持全文检索

### 倒排索引

辅助表（Auxiliary Table）中存储了单词以及单词自身在一个或多个文档中的位置间的映射（一般使用关联数组实现）

提高性能，InnoDB中使用6张Auxiliary Table，每张表使用 word 的 Latin编码进行分区

另外还使用FTS Index Cache（全文检索索引缓存）提高检索效率，红黑树结构，依据（word, list）排序

### 全文检索

match()···against()

#### Natural Language

#### Boolean

Boolean 模式查询

#### Query Explansion

扩展查询

# 锁

lock与latch，latch轻量级锁（mutex互斥锁， rwlock读写锁），lock面向事务（锁数据库中的对象，表，页和行）

## 分类

### 行锁

S Lock：共享，允许事务读一行数据

X Lock：排他，允许事务删除或者更新一行数据

### 表锁

IS Lock：意向共享锁，事务想要获取某几行的共享锁

IX Lock：意向排他锁，事务想要获取某几行的排它锁

### 一致性非锁定读

consistent nonlocking read，使用多版本并发控制获取当前执行时间数据库中的行的数据，若正在执行Update或Delete，会获取到一个快照数据（使用undo段实现）

不同事务隔离级别情况下READ COMMITTED，REPEATABLE READ情况下使用非锁定一致性读，但快照要求不一样

- READ COMMITTED------最新快照
- REPEATABLE READ------事务开始的行数据版本

### 一致性锁定读

InnoDB提供的操作

- select...for update
- select...lock in share mode

### 自增长&锁

外键&锁

对于外键，若没有加索引，则会自动增加一个索引避免表锁

## 算法

### 行锁

- Record Lock：单记录锁
- Gap Lock：间隙锁，不包含记录本身
- Next-Key Lock：Gap Lock+ Record Lock实现对于一个范围的加锁，并锁定记录本身

Record Lock：单记录锁总是锁索引记录，若没有索引，则使用隐式主键锁定

Next-Key Lock：锁定区间，技术称为Next-Key Locking，是谓词锁的一种改进，设计目的为了解决Phantom Problem(幻像问题)

​	若索引有3个值：3，7，9，则锁定区间是(-oo, 3], (3, 7], (7, 9], (9, +oo)

另外还有（Previous-Key Locking)

​	以上述为例，锁的区间是：(-oo, 3), [3, 7), [7, 9), [9, +oo)

### Phantom Problem

就是所谓的幻读，同一个事务下同一个操作执行两次结果却不相同

## 锁问题

### 脏读

### 不可重复读

### 丢失更新

### 阻塞

事务之间的阻塞

### 死锁

## 锁升级

# 事务

ACID，原子，隔离，一致和持久

## 分类

- 扁平事务
- 带保存点的扁平事务
- 链事务
- 嵌套事务
- 分布式事务

### 扁平事务

所有操作一个级别，操作都是原子的，共进退

### 带保存点的扁平事务

允许回滚到任意保存点的状态，类似存在存档的恢复，系统崩溃时，保存点会丢失

commit并不影响获取到的锁

### 链事务

保存点模式的变种，思想是在提交一个事务时，释放不需要的数据对象，将必要的处理上下文交给下一个事务（提交事务操作和下一个事务操作将合并为一个原子操作）

回滚只能到当前事务保存点状态

commit后释放所有锁

### 嵌套事务

InnoDB原生不支持

顶层事务控制着各层子事务

### 分布式事务

## 实现

原子，一致，持久------redo log和undo log实现

redo恢复提交事务修改的页操作，undo回滚行记录到某个特定版本

redo通常是物理日志，undo是逻辑日志，记录行操作

### redo

ACID------D持久

组成：内存中的重做日志缓冲（redo log buffer），重做日志文件（redo log buffer）

redo 保证事务一致性，undo帮助事务进行回滚和MVCC

redo log顺序写，undo log随机写

为确保日志写入文件，每次重做日志缓冲写人缓冲后进行一次fsync，允许用户设置磁盘刷新策略

innodb_flush_log_at_trx_commit = {0，1(default)， 2}控制重做日志刷新到磁盘

- 0------事务提交时不写入重做日志操作
- 1------事务提交时触发fsync操作
- 2------事务提交时将重做日志写入日志文件，但只写入文件系统缓存

#### log block

从左日志文件使用log block存储，512字节

#### log group

无实际存储，逻辑上的概念

#### 重做日志格式

redo_log_type | space | page_no | redo log body

从做日志类型，表空间ID，页偏移量

存储管理基于页，故日志也是基于页的

#### LSN

Log Squence Number 日志序列号，8字节，单调递增

- 重做日志写入量
- checkpoint位置
- 页的版本

#### 恢复

物理日志，恢复比undo快

### undo

只是将数据库逻辑的恢复到原貌

存储管理采用段的方式

存储引擎有rooback segment，每个段记录了1024个 undo log segment，在每个段中进行undo页申请

### purge

### group commit

## 事务控制语句

# 恢复

# 性能调优
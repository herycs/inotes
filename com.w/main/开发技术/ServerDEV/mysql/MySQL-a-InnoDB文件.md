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

```sql
       以下为索引字段信息
       ---------主键索引信息
        Table: t
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 2
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
	  ----------非主键索引信息
       Table: t
   Non_unique: 1
     Key_name: tidx
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: 3
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
```

其中字段解释分别是：

> - Table：
> - Non_unique: 1
> - Key_name: tidx
> - Seq_in_index: 索引列的位置
> - Column_name: 索引列名称
> - Collation: 列以什么方式存储在索引中，A or NULL
>     - B+树索引总是A(排序的)
>     - 使用了Heap存储引擎并建立了Hash索引则会显示NULL，不是排序的
> - Cardinality: 索引中唯一值的数目估计值
> - Sub_part: 是否列的部分索引，例如前100个数据索引则是100，全部数据则是NULL
> - Packed: 关键字如何被压缩
> - Null: 是否索引的列含有NULL
> - Index_type: BTREE，索引类型
> - Comment:注释
> - Index_comment:索引注释

## 分类

InnoDB中分为聚簇索引和辅助索引，内部都是B+树，区别是叶子节点存储到是否是一整行

### 聚簇索引

实际表依照B+树进行排序，每张表只能拥有一个聚集索引

### 辅助索引

叶子节点不包含全部行数据，除了包含键值外还有一个书签，书签告诉存储引擎在哪可以查到数据（由于InnoDB是索引组织表，故书签就是相应的**聚簇索引键**）

查数据流程：遍历辅助索引查找目标值（得到聚簇索引键），查找聚簇索引（得到实际数据）

## Cardinality

值的更新：

通过采样的方式

- 表中1/16的数据已经发生改变
- 等其他情况，innodb_stats_sample_pages设置每次采样页的数量（V1.2以前使用，default 8)

索引中唯一值的数目的估计值，用于在优化器选择时提供参考，决定是否采用此索引，但这个值一般不是每次都更新，所以存在一定的误差

当一个列的数据具有高度选择性时，就适合建立索引

>  当这个值为NULL，可能导致索引建立了却没有使用，因此系统运行一段时间后建议使用analyze table

## Fast Index Creation

> 原表的alter过程：建立新的临时表---》原数据导入---》原表删除---》新表更名为原表

快速索引创建方式------**限定于辅助索引的创建**

辅助索引的创建，对表加S锁，创建过程中不需要重建表

辅助索引删除，存储引擎更新内部视图，将辅助空间的内存标记为可用，同时删除数据库内部视图上对该表的索引定义即可

优点：避免创建临时表

缺点：由于创建过程加了S锁只能对表进行读取

## Online Schema Change

在线执行DDL，FaceBook实现的在线DDL方式

事务创建过程中可以有事务对表进行读取等操作

> 要求修改的列有主键
>
> OSC过程中允许不写入bin，所以主从同步可能有问题

## Online DDL

索引创建同时允许其他insert, delete，update操作

通过选择参数，可选择修改表的方式（原有的拷贝方式或者其他）

Lock 可以设定索引创建时对表的加锁情况{none, share, exclusive, default}

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

对每个分词都对应一个DOC_ID和Position，除此之外还有FIRST_DOC_ID，LAST_DOC_ID和DOC_COUNT

另外对于索引删除的操作

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
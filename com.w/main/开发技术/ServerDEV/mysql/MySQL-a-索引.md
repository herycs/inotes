# 索引

## 结构

B+ 树：B+树由B树 + ISAM（索引顺序访问算法）演化而来

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

（有主键依据主键建立，使用自动生成的主键列创建，若二者均不可以，则创建虚拟聚簇索引）

其检索过程本身是在多个页之间进行跳转，知道查找到具体数据

> 数据页上存储的是完整的行记录，非数据页中存放的是键值以及指向页的偏移量
>
> InnoDB本身是稀疏的通过，Recorder Header最后两个字节判断吓一跳记录位置
>
> 索引页上存储的是键值和数据页的偏移量，而不是完整行记录

### 辅助索引

叶子节点不包含全部行数据，除了包含键值外还有一个<u>书签</u>，书签告诉存储引擎在哪可以查到数据（由于InnoDB是索引组织表，故书签就是相应的**聚簇索引键**）

查数据流程：遍历辅助索引查找目标值（得到聚簇索引键），查找聚簇索引（得到实际数据）

### 关系

辅助索引负责快速定位到主键，聚簇索引负责快速得到完整行记录数据

### 索引分裂

这是索引实现最难的一部分

MySQL中的索引分裂并不总是从中间开始分裂

存储引擎会依据Page header 中的

- PAGE_LAST_INSERT
- PAGE_DIRECTON
- PAGE_N_DIRECTION

确定分裂向左还是右

分裂那个节点

- 插入随机，则分裂中间记录
- 往同一个方向的插入记录数为5，定位到插入位置之后还有3条记录，则分裂点定位到记录后的第三条记录，否则分裂点就是待插入记录

## Cardinality - 目标值的选择性

当一个列的数据具有高度选择性时，就适合建立索引

**建立索引的必要性**：对于索引的使用一般是对于少部分数据选取时建议使用，当对数据的选择性不高时没有必要使用索引，例如：sex 很可能数据行占一半，这就没有必要

**含义**：表示索引中不重复记录的数据量，实际使用中：Cardinality / Rows 应当尽可能接近1

**Cardinality值的统计**：各个存储引擎对B+的实现不尽相同，所以这个值交由存储引擎维护，通过采样的方式进行统计 

**值的更新**：通过采样的方式，触发情况执行 update & insert 操作

- 表中1/16的数据已经发生改变
- 等其他情况，innodb_stats_sample_pages设置每次采样页的数量（V1.2以前使用，default 8)

索引中唯一值的数目的估计值，用于在优化器选择时提供参考，决定是否采用此索引，但这个值一般不是每次都更新，所以存在一定的误差

采样过程：

1. 统计B+叶子节点数量 = A

2. 随机取B+树8个叶子节点，统计每个页不同节点数量 = P1，P2...P8

3. 得出预估值：
    $$
    Cardinality = \frac {\sum_{i=1}^8{P_i}} 8A
    $$
    

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

对于索引的使用需要基于具体情况具体分析

对于OLTP这种而言，一般每次的检索量少，1条或者10条这种量级，建立索引用处还是比较大的

对于OLTP在线分析应用而言，需要访问大量的记录，但是一般又有多表联查这样的情况，例如：营销额这种的销售分析，故使用索引一定程度上也可以提高查询效率

### 联合索引

MySQL可以使用多列索引，但是一般对于多列而言，例如（a，b），只有第一个列a是排序的，这时使用b为查询条件，由于b并没有排序，其实是不会走这个多列索引的

联合索引好处：

1. 多个字段定位
2. 后续列可排序

一个字段定位不到，需要多个字段辅助定位，B+树结构中的节点键值数量[>=2]

### 覆盖索引

covering index

从辅助索引中就可以查询到记录，无需再次从聚簇索引中查找，且这种索引远小于聚簇索引

最早在InnoDB Plugin中完成处理

### 优化器的选择

访问的数据占整个表很大一部分时（20%左右）会使用聚簇索引，而放弃其他索引

多表联查时，连接查询时也很容易不会使用索引

### 索引提示

Index Hint可以显示告知优化器使用的索引

使用场景：1.优化器选择错误；2.优化器可选索引很多，导致在选择上花费较多时间

### Multi-Range Read优化

5.6开始支持，目的使用此优化减少磁盘的IO，随机改为顺序

### Index Condition Pushdown优化

5.6开始支持，在取出数据同时进行where条件过滤（将过滤在存储引擎层执行了，减少了上层SQL层对数据的索取）

## Hash索引

> 一般有乘法散列，除法散列和全域散列，数据库中一般采用除法散列

InnoDB中hash -> 链表

除法散列，缓冲池中的Page页使用一个Chain指针指向相同Hash值的页

InnoDB存储引擎对于其中的页如何进行查找的呢？

表空间有个space_id，用户查找的表空间为某个16KB的页offset

存储引擎将space_id左移20位 + space_id + offset，从而定位到槽中

### 自适应Hash

存储引擎自己控制，默认为开，用户可调控

## 全文索引

InnoDB 1.2.x开始，存储引擎支持全文检索

对每个分词都对应一个DOC_ID和Position，除此之外还有FIRST_DOC_ID，LAST_DOC_ID和DOC_COUNT

另外对于索引删除的操作

### 倒排索引

以词为关键信息，建立索引表

分类：

- inverted file index->{单词，单词所在文档ID}
- full inverted index-{单词，{单词所在文档ID，文档中内容}}

辅助表（Auxiliary Table）中存储了单词以及单词自身在一个或多个文档中的位置间的映射（一般使用关联数组实现）

提高性能，InnoDB中使用6张Auxiliary Table，每张表使用 word 的 Latin编码进行分区

另外还使用FTS Index Cache（全文检索索引缓存）提高检索效率，红黑树结构，依据（word, list）排序

### 全文检索

#### InnoDB

v1.2.x开始支持，使用full inverted index，字段：word | ilist{DocumentID，Position}

由于其有position，故可以进行Proximity Search，但MyISAM不支持

倒排索引表Auxility Table，为提高性能共提供6张表，目前依据word的latin编码分区

FTS Index Cache红黑树结构，依据{word，ilist}排序

<u>更新过程</u>：先更新FTS ,再将多次更新合并到Auxillity Table

<u>查询过程</u>：时先将FTS中word字段合并到Auxility Table再查询

<u>DB关闭</u>：FTS Index Cache更新到磁盘Auxility Table中

<u>DB宕机</u>：重启后重新分词并将结果放到FTS Index Cache

<u>FTS DocumentID</u>

MySQL中为了支持全文检索，必须有一个列与word进行映射

列：FTS_DOC_ID 类型：BIGINT UNSIGNED NOT NULL 索引：FTS_DOC_ID_INDEX

<u>stop word</u>：MySQL提供stop word也就是常说的停用词的表用于避免对无用词进行统计

#### 查询

match()···against()

##### Natural Language

##### Boolean

Boolean 模式查询

##### Query Explansion

扩展查询

## 问题

1.非聚簇索引的列值一定是惟一的吗？

是，若是创建unique则必须是唯一的，否则无法定位

2.可以建立聚簇索引吗？

InnoDB自己搞定

3.为什么使用索引组织表？

堆表上的聚簇索引有时比非聚簇索引组织表更加的快

MySQL采用非聚簇索引，个人观点是

- OLAP在线分析处理应用
- 数据是否需要更新，地址是否需要调整
- 排序以及范围查找，索引组织表通过B+中间节点就能查找到所有页，然后进行读取，堆表特性决定这是不能实现的
- 非聚簇的离散度读取存在慢的情况，但数据库一般采取预读，在执行前就会预估查询量的大小从而确定是否使用索引
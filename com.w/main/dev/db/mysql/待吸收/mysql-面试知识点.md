# 这些年，面试官在MySQL上挖的那些坑

### 1、表设计上的坑

#### 	1、字段设计

​			**字段类型设计**：

​					尽量使用整型表示字符串：

​					`INET_ATON(str)`，address to number

​					`INET_NTOA(number)`，number to address	

​			**定长和非定长数据类型的选择**：

​					1、decimal不会损失精度，存储空间会随数据的增大而增大。double占用固定空间，较大数的存储会					损失精度。

​					2、定长`char`，非定长`varchar、text`（上限65535，其中`varchar`还会消耗1-3字节记录长度，而					`text`使用额外空间记录长度）

​			**尽可能使用not null**

​					1、非null字段的处理要比null字段的处理高效些！且不需要判断是否为null。
​					2、null在MySQL中，不好处理，存储需要额外空间，运算也需要特殊的运算符。如select null = null					和select null <> null（<>为不等号）有着同样的结果，只能通过is null和is not null来判断字段是否为					null。
​			**单表字段不宜过多，可以参考使用预留字段**

#### 	2、范式要求		

​			第一范式：字段原子性

​			第二范式：消除对主键的部分依赖

​			第三范式：消除对主键的传递依赖

### 2、存储引擎的选择

|              | **MyISAM** | **InnoDB**                 |
| ------------ | ---------- | -------------------------- |
| 索引类型     | 非聚簇索引 | 聚簇索引                   |
| 支持事务     | 否         | 是                         |
| 支持表锁     | 是         | 是                         |
| 支持行锁     | 否         | 是                         |
| 支持外键     | 否         | 是                         |
| 支持全文索引 | 是         | 是（5.6后支持）            |
| 适合操作类型 | 大量select | 大量insert、delete、update |

### 3、索引机制

#### 	1、索引介绍

​		索引是帮助 MySQL 高效获取数据的数据结构。

​		索引存储在文件系统中。

​		索引的文件存储形式与存储引擎有关。

#### 	2、索引分类：

​		mysql索引的五种类型：主键索引、唯一索引、普通索引和全文索引、组合索引。通过给字段添加索引可以提高数据的读取速度，提高项目的并发能力和抗压能力。

​		主键索引

​			主键是一种唯一性索引，但它必须指定为PRIMARY KEY，每个表只能有一个主键。

​		唯一索引

​			索引列的所有值都只能出现一次，即必须唯一，值可以为空。

​		普通索引

​			基本的索引类型，值可以为空，没有唯一性的限制。

​		全文索引

​			全文索引的索引类型为FULLTEXT。全文索引可以在varchar、char、text类型的列上创建

​		组合索引

​			多列值组成一个索引，专门用于组合搜索

#### 	3、索引的数据结构

​			hash

​			二叉树

​			B树

​			B+树

#### 	4、索引的应用场景

​			1、where查询条件尽可能将索引列设置为条件表达式

​			2、order by在查询的时候会使用索引列

​			3、join on的条件列设置为索引列

​			4、索引覆盖：查询的列如果全部建立过索引，那么会直接查询索引，而不会查询全部数据

​			5、字段要独立出现：

​					select * from user where id = 20-1;

​					select * from user where id+1 = 20;

​			6、模糊查询的时候，尽量不要将like语句匹配表达式以通配符开头

​			7、在使用状态值的时候不要创建索引

### 4、缓存设置

#### 	1、在配置文件中开启缓存

​			在[mysqld]段中配置query_cache_size：

​			0：不开启

​			1：开启，默认缓存所有，需要在SQL语句中增加select sql-no-cache提示来放弃缓存

​			2：开启，默认都不缓存，需要在SQL语句中增加select sql-cache来主动缓存（常用）

#### 	2、在客户端设置缓存大小

​			通过配置项 set global query_cache_size=64x1024x1024

### 5、数据分区

​	当数据量较大时，MySQL的性能就开始下降，需要将数据分散到多组存储文件中，以保证单个文件的执行效率

​	分区算法：

​		1、hash(field)	适用于整型字段

​		2、key(field)	适用于字符串类型

​		3、range算法	按照数据大小范围分区	

```sql
create table article_range(
    id int auto_increment,
    title varchar(64),
    content text,
    created_time int,   -- 发布时间到1970-1-1的毫秒数
    PRIMARY KEY (id,created_time)   -- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY RANGE(created_time)(
    PARTITION p201808 VALUES less than (1535731199),    -- select UNIX_TIMESTAMP('2018-8-31 23:59:59')
    PARTITION p201809 VALUES less than (1538323199),    -- 2018-9-30 23:59:59
    PARTITION p201810 VALUES less than (1541001599) -- 2018-10-31 23:59:59
);
```

​		4、list算法

```sql
create table article_list(
    id int auto_increment,
    title varchar(64),
    content text,
    status TINYINT(1),  -- 文章状态：0-草稿，1-完成但未发布，2-已发布
    PRIMARY KEY (id,status) -- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY list(status)(
    PARTITION writing values in(0,1),   -- 未发布的放在一个分区   
    PARTITION published values in (2)   -- 已发布的放在一个分区
);
insert into article_list values(null,'mysql优化','内容示例',0);
flush tables;
```

### 6、集群

#### 	1、主从复制

​		mysql主从是异步复制过程

​		1、master开启bin-log功能，日志文件用于记录数据库的读写增删
​		2、需要开启3个线程，master IO线程，slave开启 IO线程 SQL线程，
​		3、Slave 通过IO线程连接master，并且请求某个bin-log，position之后的内容。
​		4、MASTER服务器收到slave IO线程发来的日志请求信息，io线程去将bin-log内容，position返回给slave IO线程。
​		5、slave服务器收到bin-log日志内容，将bin-log日志内容写入relay-log中继日志，创建一个master.info的文件，该文件记录了master ip 用户名 密码 master bin-log名称，bin-log position。
​		6、slave端开启SQL线程，实时监控relay-log日志内容是否有更新，解析文件中的SQL语句，在slave数据库中去执行。

#### 	2、读写分离

​		读写分离是依赖于主从复制，而主从复制又是为读写分离服务的。MySQL读写分离是指让master处理写操作，让slave处理读操作，非常适用于读操作量比较大的场景，可减轻master的压力

#### 	3、负载均衡

​		轮询

​		加权轮询

​		负载分配

#### 	4、高可用

​		在服务器架构时，为了保证服务器7x24不宕机在线状态，需要为每台单点服务器（由一台服务器提供服务的服务器，如写服务器、数据库中间件）提供冗余机。

​		对于写服务器来说，需要提供一台同样的写-冗余服务器，当写服务器健康时（写-冗余通过心跳检测），写-冗余作为一个从机的角色复制写服务器的内容与其做一个同步；当写服务器宕机时，写-冗余服务器便顶上来作为写服务器继续提供服务。对外界来说这个处理过程是透明的，即外界仅通过一个IP访问服务。

### 7、SQL语句优化

#### 	1、数据库导入语句

​		1、导入时先禁用索引和约束：

​			alter table table-name disable keys

​			待数据导入完成之后，再开启索引和约束，一次性创建索引

​			alter table table-name enable keys

​		2、数据库如果使用的引擎是innodb，那么它默认会给每条写指令加上事务（这也会消耗一定的时间），因此建议先手动开启事务，再执行一定量的批量导入，最后手动提交事务。
​		3、如果批量导入的SQL指令格式相同只是数据不同，那么你应该先prepare预编译一下，这样也能节省很多重复编译的时间。

#### 	2、limit offsets,row

​		尽量保证不要出现大的offset，比如limit 10000,10相当于对已查询出来的行数弃掉前10000行后再取10行，完全可以加一些条件过滤一下（完成筛选），而不应该使用limit跳过已查询到的数据。

#### 	3、select * 要少用

​		即尽量选择自己需要的字段`select`，但这个影响不是很大，因为网络传输多了几十上百字节也没多少延时，并且现在流行的ORM框架都是用的`select *`，只是我们在设计表的时候注意将大数据量的字段分离。

#### 	4、单表和多表查询

​		多表查询：join、子查询都是涉及到多表的查询。如果你使用explain分析执行计划你会发现多表查询也是一个表一个表的处理，最后合并结果。因此可以说单表查询将计算压力放在了应用程序上，而多表查询将计算压力放在了数据库上。

#### 	5、count(*)

​		在MyISAM存储引擎中，会自动记录表的行数，因此使用count(*)能够快速返回。而innodb内部没有这样一个计数器，需要我们手动统计记录数量.

#### 	6、开启慢查询日志

​		配置项：slow_query_log

​		可以使用 show variables like ‘slov_query_log’ 查看是否开启，如果状态值为OFF，可以使用 set GLOBAL slow_query_log = on来开启，它会在datadir下产生一个xxx-slow.log的文件。

### 8、MySQL配置

​	1、max_connections		最大客户端连接数

​	2、table_open_cache		表文件缓存

​	3、key_buffer_size			索引缓存大小		

​	4、innodb_buffer_pool_size	innodb存储引擎缓存池大小

​	5、innodb_file_per_table	将每一个表的数据放在一个文件中而不是共享表空间
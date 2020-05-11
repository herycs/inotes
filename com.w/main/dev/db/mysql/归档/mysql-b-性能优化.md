# 性能优化

## 优化数据库服务器

优化服务器硬件

修改my.ini文件

控制台优化

查询主要性能参数

```sql
show status like 'value';

show variables like 'value';
```

设置性能指标参数

```sql
1.查看变量
show variables like '%query_cache%';

2.使用set 语句修改值
示例：set @@global.query_cache_limit = 67108864b;
```

## 优化查询

### 分析查询语句

 explain 语句

```sql
explain select statement;
```

describe语句

```
describe select statements;
```

### 使用索引优化查询

应用like关键字优化索引查询

查询语句中使用or关键字

查询语句中使用多列索引

索引字段上使用函数操作

### 优化多表查询

## 优化数据结构

### 优化表结构

字段很多的表分解为多个表

增加冗余字段

合理设置表的数据类型和属性

### 增加中间表

### 优化插入记录速度

### 分析表，检查表和优化表

- 利用analyze语句分析表

- 使用check 语句检查表

- 使用optimize优化表

### 优化慢查询

### 优化表设计

## 查询高速缓存

检验高速缓存是否开启

```sql
show variables like '%query_cache';
```

使用高速缓存

```sql
使用set命令设定有关值即可
```

## 优化性能的其它方面

limit 1:查询结果只有1行时可以提升速率

少用select * from table_name，尽量明确字段

不滥用sql自动转换

避免在where子句中对null判断

避免在where子句中使用“！=”或"<>"操作符

避免where子句对字段进行表达式操作

避免使用in或not in
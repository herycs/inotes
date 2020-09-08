# 自增ID

## 语句

```mysql
+------------+--------------+------+--------+
| id         | name         | age  | addr   |
+------------+--------------+------+--------+
| 2147483646 | max_id       |  124 | tess   |
| 2147483647 | NULL         |   11 | rr     |
```

insert into user (age, addr) value(11, 'tr');
ERROR 1062 (23000): Duplicate entry '2147483647' for key 'PRIMARY'

## 因对策略

1. 设计

    考虑到数据量规模，选取合适的数据类型

    *bigint unsigned* 

2. 处理跳跃ID

    当前的数据除ID外拷贝至新表中，并再拷贝完后删除当前表，修改新表表名，完成替换
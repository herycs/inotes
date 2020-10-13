# 死锁

### 不同表相同记录行锁冲突

事务AB分别操作表1， 表2

| 时间 | tx_A  | tx_B  |
| ---- | ----- | ----- |
|      | 锁a表 |       |
|      |       | 锁b表 |
|      | 锁b表 |       |
|      |       | 锁a表 |

--->相互等待



### 相同表记录行锁冲突

| 时间 | tx_A     | tx_B     |
| ---- | -------- | -------- |
|      | 锁id = 1 |          |
|      |          | 锁id = 2 |
|      | 锁id = 2 |          |
|      |          | 锁id = 1 |

--->相互等待



A事务加锁序列：1 3 4 2 5

B事务加锁序列：3 1 4 2 5

上述加锁序列中，当A加锁1时没问题，B加锁3时没问题，但是当B加锁1时，A加锁3时会发现对方持有了锁

```mysql
事务A:
mysql> update user set age = 333 where id = 3;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> update user set age = 666 where id = 4;
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction -- 死锁
```



```mysql
事务B:

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update user set age = 444 where id = 4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> update user set age = 555 where id = 3;
Query OK, 1 row affected (18.97 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

### 不同索引锁冲突

| 时间 | tx_A                                | tx_B     |
| ---- | ----------------------------------- | -------- |
|      | 锁索引a，（锁索引同时会加聚簇索引） |          |
|      |                                     | 锁id = 1 |

--->相互等待



以下序列加锁主要发生于对索引加锁情况

A事务加锁序列：1 3 4 2 5

B事务加锁序列：3 1 4 2 5	

上述加锁序列中，当A加锁1时没问题，B加锁3时没问题，但是当B加锁1时，A加锁3时会发现对方持有了锁

```mysql
事务A：
mysql> update user set name = '111' where age = 12;
ERROR 1062 (23000): Duplicate entry '111-12' for key 'na'
mysql> update user set age = 444 where name > 'tt';
Query OK, 12 rows affected (0.00 sec)
Rows matched: 12  Changed: 12  Warnings: 0

mysql> update user set age = 444 where name > 'tt';
Query OK, 0 rows affected (0.00 sec)
Rows matched: 12  Changed: 0  Warnings: 0
```

```mysql
事务B：
mysql> update user set age = 123 where id > 20;
Query OK, 4 rows affected (9.63 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```

原因：操作语句不同，但是操作其他非主键字段，同时也会向聚簇索引加锁，而这种情况下等同于在通过ID加锁

### gap锁冲突



## 如何尽可能避免死锁

1）顺序——顺序访问表行

比如对第2节两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；又比如对于3.1节的情形，将两个事务的sql顺序调整为一致，也能避免死锁。

2）事务粒度——大事务拆小

大事务里的加锁很多，容易触发死锁，事务拆分小可尽可能避免这种情况

3）锁粒度——一次锁定

一次锁定所有资源，从而避免AB事务交替性加锁，导致相互等待

4）隔离级别——降低隔离级别

如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。

5）使用索引——为表添加合理的索引

可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。

## 如何定位死锁成因

1）通过应用业务日志定位到问题代码，找到相应的事务对应的sql；

2）确定数据库隔离级别。

3）找DBA执行下show InnoDB STATUS看看最近死锁的日志。



## ###

RR级别：

update 操作同一个id，或者同一个属性值所代表的行，事务阻塞

执行结果：A，B事务的查询结果如下

```mysql
-- tx_A: set age = 111;
 8 | tt           |  111 | xian
-- tx_B: set age = 222;
 8 | tt           |  222 | xian
```

update一个已被删除的id，则执行效果是0 rows affected，0 rows matched

delete同一个id，事务阻塞，且包括对此数据的非事务类型删除操作也被阻塞了

select一个在其他事务中已被删除的id，可以查询到值

### 插入语句主键冲突怎么处理

1.使用ingore

insert ingore [table_name] (col_1, col_2) values(val_1, val_2);

若存在这个id则跳过这条命令。


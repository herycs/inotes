# 命令备忘录

## 查看IOThread

show engine innodb status;

# Linux操作数据库

# 开关数据库

```mysql
-- 启动
service mysqld start;
service mysql start;
/etc/inint.d/mysqld/ start
safe_mysqld&
-- 关闭
service mysqld stop;
/etc/init.d/mysqld stop
mysqladmin shutdown;
-- 重启
service mysqld restart;
service mysql restart;
/etc/init.d/mysqld restart
```

# 数据库操作

## 创建数据库

```sql
create database if not exists school default charset utf8mb4;
```

## 分区

```sql
create table con(
	id int,
	a int,
	b int,
	c char(3)
	)Engine = InnoDB 
	partition by range columns(id)(
		partition p0 values less than (10),
		partition p1 values less than (20));
```

上述range分区建立，表将不再由一个*.ibd文件构成，而是分成con#P#p0.ibd，com#P#p1.ibd

```sql
create table con(
	id int,
	a int,
	b int,
	c char(3)
	)Engine = InnoDB 
	partition by list (id)(
		partition p0 values less than (10),
		partition p1 values less than (20));
```

上述list分区建立，使用list，分区列值是离散的，而非连续的

这种分区情况下，向分区中插入未定义的值MyISAM会将之前的row数据插入，而InnoDB会将其视为一个事务（也就是没有数据插入）

```sql
create table con(
	id int,
	a int,
	b int,
	c char(3)
	)Engine = InnoDB 
	partition by hash (id)(
		partition p0 values less than (10),
		partition p1 values less than (20));
```

子分区：InnoDB运行在List和Range上再使用List或Hash分成子分区关键字，subpartition

分区的Null值，List分区情况下必须显示指定Null存放的分区，Range分区情况下优先放到靠前空间（若以0,1,2,3等顺序命名分区就是指编号较小空间）,Hash分区情况下任何含有NULL值的记录都会返回为0

## 索引

索引更新命令

analyze table

# 持久化

## 导出文件

- 命令：mysqldump -h 地址 -P 端口 -u 用户名 -p 密码  数据库名 > 导出后保存到的地址及文件名

- 示例

    ```mysql
    -- 命令1 ：以下语句可以可导出文件（若你的用户所拥有的权限够的话），但会有警告
    mysqldump -h 192.168.1.1 -P 3306 -u root -p 123456 db1 > d:\db1.sql
    
    -- 命令2 避免报警
    mysqldump -h 192.168.1.1 -P 3306 -u root -p db1 > d:\db1.sql
    ```

# 事务

```
set @@autocommit = 0;
select @@tx_isolation;

set session transaction isolation level read uncommitted;
set session transaction isolation level read committed
set session transaction isolation level repeatable read
set session transaction isolation level serializable

start transaction;

select * from stus;
```

# 锁

## 查询请求信息

数据库锁信息

show engine innodb status\G;

show full processlist;



select * from information_schema.innodb_trx\g;

select * from information_schema.innodb_locks\g;

select * from information_schema.innodb_lock_waits\g;

# 主从注意事项

对于自增长列，InnoDB需要考虑自增长并发情况，对于MyISAM而言是表锁无需考虑并发情况


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


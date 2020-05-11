# 日志文件管理

## 错误日志

启用和设置错误日志

```sql
将log-error添加到my.ini文件的[mysqld]组中
# my.ini
log-error=[path/[filename]]

-- path:日志文件路径
-- filename:日志文件名
```

查看错误日志

```sql
show variables like 'log_error';
```

删除错误日志

```sql
mysqladmin -u root -p flush - logs
```

## 二进制日志

启用二进制日志

```sql
my.ini文件的log-ini文件的log-bin选项可以开启二进制日志
#my.ini文件
[mysqld]
log-bin[=path\[filename]]
expire_logs_days = 10
max_binlog_size = 100M
```

查看二进制日志

```sql
show binary logs;

mysqlbinlog filename.number
```

清理二进制日志

```sql
删除所有
reset master;

删除指定的日志文件
purge master logs语句
格式：
purge {binary|master} logs to 'log_name'
purge {binary|master} logs before 'date'
```

二进制日志恢复数据库

```sql
mysqlbinlog [option]filename|mysql -u user -p password
```

暂时停止二进制日志功能

```sql
set sql_log_bin = {0|1}
```

## 通用查询日志

启动和设置通用查询日志

```sql
#my.ini文件
[mysqld]
log[=path\[filename]]

不指定参数
[mysqld]
log
```

查看通用日志

```sql
日志以文本文件存储，使用文本文件查看器查看
```

删除通用查询日志

```sql
开启新通用日志可覆盖之前的日志
mysqladmin -u root -p flush -logs
```

## 慢查询日志

启用慢查询

```sql
#my.ini
[mysqld]
log - slow - queries[=path\[filename]]
long_query_time = n
```

操作慢查询日志

删除慢查询日志

```sql
mysqladmin -u root -p flush - logs
```


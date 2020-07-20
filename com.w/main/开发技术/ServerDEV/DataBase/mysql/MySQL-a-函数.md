# MySql基本信息查询

> 命令限制：
>
> - 大小写不敏感
> - 一行可书写多个命令，以分号隔开
> - 提示符<kbd>></kbd>变为<kbd>-></kbd>，表明MySQL在等待输入完整

## 获取信息

查询服务器版本号：

```sql
SELECT VERSION();
```

查询当前日期：

```sql
SELECT　CURRENT_DATE;
SELECT NOW();
```

查询用户：

```sql
SELECT USER();
```

## 基本字符集

查看

> 查看所有字符集

```sql
show character set;
```

> 查看校对原则

```sql
show collation like 'latin1%';
```

## 查询辅助

### ifnull

```sql
//若为空则返回后面的null
select 
    ifnull ((select distinct Salary from Employee order by Salary desc limit 1 offset 1),null)
    as SecondHighestSalary;
```

### limit

```sql
limit n, m
限制返回第n个位置到m个位置间的数据
索引从0开始，所以想要第n个需要使用n-1
```

### offset

```sql
当前位置偏移几个
-- 偏移n个
offset n
```

## 内置函数

### 时间

```sql
select adddate('1-1-1', '180') as 'Date' ;
select addtime('13:25:44', '1:2:1') as 'Time';
select current_time() as 'curr time';
select current_date() as 'curr data';
select current_timestamp() as 'curr time stamp';
select current_user() as 'who am i';
-- 日期差
select datediff(date('2022-01-02'), current_date());
```

### 字符串

### 数学计算
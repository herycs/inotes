# 备份与恢复

## 备份，恢复概述

故障原因

- 系统故障
- 事务故障
- 介质故障

备份分类

按备份时服务器是否在线分

- 热备份
- 温备份
- 冷备份

备份内容分

- 逻辑备份
- 物理备份

备份涉及的数据

- 完整备份
- 增量备份
- 差异备份

备份时机

- 创建数据库，添加数据后
- 创建索引后
- 清理事务日志后
- 执行无日志操作后

恢复方法

- 数据库备份
- 二进制日志
- 数据库复制

## 数据备份

mysqldump命令

备份数据库或表

```sql
mysqldump -u user -h host -p password
	-- databasename[all -databases][tablename= ,[tablename= ...]] > filename.sql
```

备份多个数据库

```sql
mysqldump -u user -h host -p 
	-- databases databasename[ databasename...]
-- 多个数据库名空格隔开
```

直接复制整个数据库目录

mysqlhotcopy工具快速复制

```sql
[root@localhost ~] # mysqlhotcopy [option] dbName1 dbName2 ... backupDir/
```

## 数据恢复

执行相应的备份文件

```sql
mysql -u user -p[databasename]<filename.sql;
```

使用source恢复表和数据库

恢复表

```sql
mysql命令行:
1.进入数据库
2.执行命令source d:/mysqlDemo1.sql
```

恢复数据库

```sql
1.进入数据库
2.执行source filename.sql
```

## 数据库迁移

相同版本数据库迁移

> 迁移前后数据库主版本不同，复制数据库目录（只有数据库表都是MyISAM时才可以）

不同版本的数据库之间的迁移

低版本迁移至高版本

MyISAM：直接复制或者mysqlhotcopy工具

InnoDB：使用mysqldump命令实现

高版本迁移

建议使用mysqldump命令实现

不同类型数据库迁移

> 没有普遍的实现方法

数据库迁移至新服务器

```sql
mysqldump -u username -p password databasename | mysql - host = hostname -c databasename
--示例：
mysqldump -h 192.168.1.1 -P 3306 -u root -p db1 > d:\db1.sql

-- mysqldump其它选项:
-- -add-locks:每个表导出之前添加lock tables,之后unlock table
-- -add-drop-tabls:在每个create语句之前增加一个drop table
-- -allow-keywords:允许创建是关键字的列名字
-- -c, -complete-insert:使用完整的insert语句（用列名）
-- -c, -compress:客户服务器都支持压缩，压缩两者间的所有信息
-- -delayed:用insert delayed命令插入行
-- -e, -extend-insert:使用全新多行insert语法（给出更紧缩更快的插入语句）
```

## 表的导入与导出

select...into outfile导出文件

```sql
select [columnlist] from table [where codition] into outfile'filename' option
```

mysql命令导出文本文件

```sql
mysql -u root -pPassword -e| -- excute = "select statement" databasename > c:\name.txt;
```

load data infile

```sql
load data[low_prioriy|concurrent]
	infile'filename.txt'[replace|ignore] 
		into table tbalename [options] [ignore number lines] 			[{columnname[|uservariables],..}]
			[set columnname = exression,..]
			
-- low_prioriy|concurrent:指定low_prioriy延迟语句的执行
-- filename.txt:该文件中保存了待存入数据库的数据行
-- replace|ignore:指定了replace,当文件中出现与原有行相同的唯一关键字值时，输入航替换原有行
```

# 
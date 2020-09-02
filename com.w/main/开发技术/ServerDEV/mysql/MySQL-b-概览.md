# MySQL概览

# 基础类型

数据类型

运算符、表达式

常用函数

# 数据检索


# 基础操作

## 基础编程

自定义变量

- 定义用户变量set或select

> 使用区别：
>
> set @value_name = (select * from table__name);
>
> select @ value_name := (select * from table_name);

局部变量

- declare 声明局部变量

> 示例：
>
> declare value_demo1 int default 100;

- 局部变量的范围仅限于存储程序中（函数，触发器，存储过程等），且在使用时不可将名称定义为表字段名，否则得不出预期结果

定界符delimiter

> 为了解决遇到 ; 语句就执行的情况

- 使用语法：delimiter 定界符 查询语句 定界符

  示例：delimiter // select * from table_1 //

begin...end语句块

- 使用：

  begin 

  ​	[局部]变量声明;

  ​	 程序代码行集;

   end

预处理

基本语句

- prepare
- execute
- deallocate

使用步骤

- 创建
- 使用
- 释放

## 自定义函数

### 创建函数

> 修改结束认定符以避免函数返回语句后的分号被认定为结束符，从而语句被执行产生错误
>
> 1.创建函数需写出函数名、函数参数、函数返回值类型
>
> 2.在begin...end语句中编写函数的具体执行内容
>
> 最终为符合常规执行sql语句的习惯，将结束认定符修改回分号

- 创建示例

```sql
1.修改结束符为其它符号
delimiter //

2.创建函数
create function function_name(val_1 int, val_2 int) returns char

begin
return val_1 * val_2;
end
3.结束创建
//
4.修改结束认定符为;
delimiter ;
```

### 使用函数

```
select function_name(12, 13);
```

### 查看函数

```sql
查看指定数据库中的所有自定义函数
show function status;(函数少时使用)
show function status like 模式;(函数多时使用)

查看指定数据库中所有自定义函数名
select name from MySQL.proc 
where db='course' and type='function';

查看指定函数名的详细信息
show create function function_name;

函数信息保存在information_schema数据库中的routines表中
检索表可查看相关函数信息
select * from information_schema.routines
	where routine_name = 'function_name';

```

### 修改函数

```sql
格式
alter function func_name[characters -]
```

### 删除函数定义

```sql
drop function func_name;
```

## 控制流语句

### if

```sql
格式（其中“[]”中为可选并非使用时要带上 []）
if condition then
...
[else condition then]
...
[else]
...
```

### case

```
格式
case value
	when vlaue then
	...
	else
	...
end case
```

### 条件判断函数

- if()

  ```sql
  if (condition , v1, v2)
  condition为true,返回 v1, 否则返回v2
  ```

- ifnull()

  ```sql
  ifnull(v1, v2)
  从v1开始，返回第一个不为null的值
  ```

- case

  ```sql
  case 表达式1
  when 值1 = 条件1 then 结果1
  when 值2 = 条件2 then 结果2
  else 其他值
  ```

## 循环语句

### while

```sql
while condition do
...
end while;
```

### loop

```sql
loop
...
end loop
```

### repeat

```sql
repeat
...
until condition
end repeat
```

# 存储过程，游标，触发器

## 存储过程

创建存储过程

```sql
存储过程有三类参数，in(输入), out(输出), inout(输入输出)
create produce name(params);
```

调用存储过程

```sql
call [db.name]sp_name(params);
```

查看定义

```sql
show produce status like 'do_%';

show create produce do;

select * from information_schema.routines where routine_name = 'do';

show create produce do;
```

条件和处理程序定义

```sql
定义条件
declare condition_name condition for condition_type;
condition_type:
sqlstate [value] sqlstate_value|mysql_error_code
-- condition_name：错误触发条件的名称
-- condition_type:条件类型，MySQL错误代码，ANSI标准错误代码
				sqlstate_value是长度为5的字符串类型错误代码
				mysql_error_code数值类型错误代码
				
定义处理程序
declare handler_type handler for condition_value[] sp_statement handler_type:
continue|exit
condition_value:
sqlstate [value]sqlstate_value|condition_name|sqlwarning|not found|sqlexception|mysql_error_code
-- handler_type: 错误处理类型
				continue发生错误不处理，执行其他语句
				exit发生错误退出sql语句的执行
-- condition_value:错误触发条件
				sqlstate[value]sqlstate_value:长度为5自UC哈UN类型错误代码
				condition_value:定义的错误条件名称
				sqlwarning:匹配所有01开头的sqlstate错误代码
				not found:匹配所有02开头的sqlstate错误代码
				sqlexception:匹配其它非sqlwarning和not found捕获的错误代码
				mysql_error_code:数值型错误代码
-- sp_statement:自定义错误处理程序
```

修改存储过程

```sql
修改参数
alter produce sp_name[characteristic ...]

修改权限
alter procedure do_name modifies sql data sql security invoker;
```

删除存储过程

```sql
drop produce[if exits] sp_name
```

## 游标处理结果集

> 流程：声明游标->打开游标->提取数据，处理数据->关闭游标

声明游标

```sql
declare cursor_name cursor for select_statement;

示例：
declare teach_cursor cursor
	for select name, age from students;
```

打开游标

```sql
open cursor_name;
```

游标中提取数据

> 注意：
>
> 变量，需要与声明时的变量名个数一致
>
> fetch语句，
>
> ​		每次只能提取一条语句，因此想便利整个结果集需要与循环语句结合
>
> ​		fetch提取完最后一条语句之后，再次提取结果集会
>
> ​				报{ERROR 1329(0200)}错
>
> 游标错误处理程序应当写在游标声明语句之后，通常两者结合结束结果集的遍历		

```sql
fetch cursor_name into var1[,var2,..]
```

关闭游标

```sql
close cursor_name
```

## 触发器

触发器建立

> 触发器执行顺序，before，执行操作，after
>
> 触发器中可以使用old,new关键字
>
> ​	old代表语句执行的前的数据只读，只可引用，不可更改
>
> ​	new代表语句执行后的数据
>
> 对于insert 只有new是合法的
>
> 对于delete 只有old是合法的
>
> 对于update old和new都合法

```sql
create trigger_name trigger_time trigger_event on table_name for each row trigger_statement

-- tbale_name:触发程序相关表
-- trigger_time:触发动作时间
-- trigger_event:激活触发器的语句类型，不支持同一个表中同时存在两个相同的激活触发的类型
	insert
	update
	delete
-- for each row:声明由触发器激活的每一行都要激活触发器的动作
-- trigger_statement:触发程序激活时执行的语句
```

触发器注意事项

- 触发器中select语句不能返回结果集
- 同一个表不能创建两个相同触发时间触发时间的触发程序
- 触发程序中不能使用显式或隐式方式打开、开始和结束事务
- 批量更新数据触发器会降低更新效率
- MyISAM中触发器不能保证原子性
- InnoDB中建议级联维护外键数据，MyISAM中使用触发器实现级联修改删除
- 使用触发器维护InnoDB 维护数据，先维护父表再维护子表
- mysql触发器中不可对本表使用更新语句，可使用set命令代替
- before 中auto_increment字段的new值为0不是插入操作自动生成的自增型字段值
- 添加触发器建议进行详细测试

删除触发器

```sql
drop trigger[schema.]trigger_name
```

## 事件及应用

认识事件调度

- 开启事件调度

  ```sql
  set @@global.event_scheduler = true;
  
  在mysql配置文件my.ini中加入 event_scheduler = 1;
  ```

- 查看事件

  ```sql
  show variables like 'event_scheduler'
  
  select @@event_scheduler
  ```

使用事件

- 创建

  ```sql
  create event [if not exists] event_name on schedule schedule [on completion[noe preserve]] [enable|disable|disable on slave] [comment'comment'] do sql_statement;
  -- event_name: 事件名
  -- schdeule:时间调度
  	at子句：事件在某个时刻发生
  		timestamp具体一个时间点
  	every子句：在指定时间区间内每隔多长时间发生一次
  		starts开始时间，ends结束时间
  -- do sql_statement:事件启动时执行SQL代码
  ```

- 管理事件

  ```sql
  show events [from schema_name] [like 'pattern'|where expr]
  
  show events;
  
  show evetns\G;
  
  show create event event_name;
  ```

- 修改事件

  ```sql
  alter event event_name [on schedule schedule]
  [rename to new_name] on completion[not] preserve]
  [comment'comment'][enable|disable][db sql_statement]
  ```

- 删除事件

  ```sql
  drop event[if exists][database name.]event_name
  ```

# 权限管理及控制

## mysql权限管理系统原理

###  mysql权限表

user表

- 查看mysql数据库提供的权限管理

  ```sql
  use mysql;
  
  desc user;
  ```

db表和host表

- db表存储用户对某个数据库的操作权限，决定用户可以从哪个主机存取某个数据库
- host表存储主机对数据库的操作权限，不受grant和revoke影响

tables_priv表和column_priv表

- tables_priv对单个表进行权限控制
- column_priv对表的操作权限

 procs_priv表

> 对存储过程及存储权限进行控制

### 权限工作原理

连接核实

> 基于用户提供信息验证用户身份

请求核实

> 得到连接许可后，对用户的所有操作权限进行检查

- 操作权限检查流程

  > 以下前5个权限检查中只要有一个权限允许则执行操作，否则执行第六步

  <kbd>收到用户请求操作</kbd>--><kbd>user表中权限</kbd>--><kbd>db和host表中权限</kbd>--><kbd>tables_priv中权限</kbd>--><kbd>column_priv中权限</kbd>--><kbd>第六步，不允许用户请求的操作</kbd>

## 账户管理

### 普通用户管理

用户创建

- create语句

```sql
- 
create user user
	[identified by [password] 'password'] 
	[,user[identified by [password]'password']] 
	[,...]

-- user格式: 'user_name'@'host_name'
-- host_name: 用户创建的使用mysql的连接来自的主机，若存在_或%则使用单引号括起来，“%”表示一组主机
-- localhost: 本地主机
-- identified by 指定用户密码，用户名密码区分大小写
-- password(): 对密码加密
-- create创建多个用户，使用,分开

示例：
create user
	'userDemo1'@'localhost' identified by '123456';

- 
不指定明文
1.获取密文
select password('123456');
-- 执行结果：
--    +-------------------------------------------+
--    | password('123456')                        |
--    +-------------------------------------------+
--    | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
--    +-------------------------------------------+
2.创建用户
create user 'userDemo2'@'localhost' identified by password '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

- grant语句

```sql
grant priv_type on database.table to user 
	[identified by[password]'password'] 
	[,user[identified by[password]'password']]
	[, ...]
	
示例：
grant 
	select, update 
		on jwgl.course 
			to 'userDemo3'@'localhost' 
				identified by '123456';
```

查看用户

```sql
select user 'userDemo1' from user;
```

删除用户

- drop

  > 删除并取消用户权限
  >
  > 要求：
  >
  > ​	使用此语句需要全局create user或delete user权限
  >
  > ​	此语句不可关闭任何打开的用户会话，若存在打开的用户会话，当会话结束后才会生效

  ```sql
  drop user user_name
  	[,user_name]
  	[,...]
  	
  示例：
  drop user 'user_name'@localhost;
  ```

  

- delete

  ```sql
  delete from mysql.user 
  	where host = 'host_name' and user = 'user_name';
  ```

修改用户名称

```sql
rename user old_name to new_name
	[,old_name to new _name]
	[,..]
```

### 登录及修改密码

登录mysql服务器

```sql
mysql -h hostname|hostIP -P port -u username -p DatabaseName -e 'SQL Statements';

-- mysql: 登录命令
-- -h hostname|hostIP: 登录主机
-- -P port: 端口，默认3306，可以省略
-- -u username:登录服务器账号名
-- -p : 密码
-- DatabaseName: 要打开的数据库名
-- -e "sql statements": 要执行的SQL语句串
-- 结束符: 以；或者 \g结束
```

修改用户密码

> 可使用mysqladmin，update或set password实现

- root用户修改自己密码

  ```sql
  1.mysqladmin
  mysqladmin -u username  -h localhost -p password "new password"
  
  2.update
  update mysql.user 
  	set password  = password('new_password') 
  		where user = 'root' and host = 'localhost';
  ```

- 修改用户密码

  ```sql
  set password [for user] = password('new_password');
  ```

## 权限管理

9.3.1权限层级

- 全局层级
- 数据库层级
- 表层即
- 列层级
- 子程序层级

授权管理

授权grant

```sql
grant priv_type[(column_list)]
	[,priv_type[(column_list)]]
	[,...n]
        on dbName.tbale_name|*.*|*.tbale_name|dbName.*
            to user[identified by [password]'password']
               [,user[identified by [password]'password']]
               [,...n]
                   [with grant option]
```

收回权限revoke

```sql
收回指定权限
revoke priv_type[(column_list)]
	[,priv_type[column_list]]
	[,...n]
		on dbName.tbale_name|*.*|*.tbale_name|dbName.*
			from 'username'@hostName
				[,'username'@hostName]
				[,...n]
收回所有权限
revoke all privilages, grant option 
	from 'username'@'hostname'
	[,'username'@'hostname']
	[,...n];
```

查看权限

```sql
show grants for 'username'@'hostname'
```

## 数据库安全常见问题

权限更改合时生效

```sql
1.管理员身份进入命令行
flush privieges;

2.操作系统中
mysqladmin flush - privileges;

3.操作系统中
mysqladmin reload;
```

账户密码设置

> 不需要password()加密密码,会自动加密
>
> ​	grant...identified by
>
> ​	mysqladmin password
>
> 需要加密，操作针对user表，user表中密码都是加密的
>
> ​	set password, insert, update

```sql
1.DOS命令窗口中指定密码

2.set password 设置密码

3.grant usage指定账户密码

4.创建新账户指定密码

5.update更新已有账户密码
```

关于mysql中匿名登录

> mysql在windows下一般创建了两个root账户

```sql
1.为root账户和匿名账户指定密码

​	set password 或 update

2.删除匿名账户
```

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

禁用索引

- 避免插入记录时对其排序，可在插入前禁用索引，插入后启用索引

禁用唯一性检查

采用insert的优选方式

> 插入多条记录

- 方法1:一条insert语句插入一条记录，执行多个insert实现多数据插入
- 方法2（推荐）:一条insert语句插入多条语句，此方式减少了和数据库的连接，速度较快

### 分析表，检查表和优化表

利用analyze语句分析表

使用check 语句检查表

使用optimize优化表

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


# 权限管理

## mysql权限管理系统原理

### mysql权限表

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

操作权限检查流程

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

drop

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

delete

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

权限层级

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

### 权限更改何时生效

```sql
1.管理员身份进入命令行
flush privieges;

2.操作系统中
mysqladmin flush - privileges;

3.操作系统中
mysqladmin reload;
```

### 账户密码设置

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

### 关于mysql中匿名登录

> mysql在windows下一般创建了两个root账户

```sql
1.为root账户和匿名账户指定密码
set password 或 update

2.删除匿名账户
```

# 
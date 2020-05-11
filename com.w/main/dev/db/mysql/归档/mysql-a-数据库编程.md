# 基础编程

## 基础概念

### 定界符delimiter

> 为了解决遇到 ; 语句就执行的情况

- 使用语法：delimiter 定界符 查询语句 定界符

    示例：delimiter // select * from table_1 //

### begin...end语句块

- 使用：

    ```sql
    begin 
        -- [局部]变量声明;
        -- 程序代码行集;
     end
    ```

### 预处理

基本语句

```sql
prepare
execute
deallocate
```

使用步骤

- 创建--->使用--->释放

## 函数

## 自定义变量

定义用户变量set或select

```sql
-- 使用示例
set @value_name = (select * from table__name);
select @ value_name := (select * from table_name);
```

局部变量

- declare 声明局部变量

    ```sql
    declare value_demo1 int default 100;
    ```

- 局部变量的范围仅限于存储程序中（函数，触发器，存储过程等），且在使用时不可将名称定义为表字段名，否则得不出预期结果

## 自定义函数

创建函数

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

使用函数

```
select function_name(12, 13);
```

查看函数

```sql
--查看指定数据库中的所有自定义函数
show function status;(函数少时使用)
show function status like 模式;(函数多时使用)

--查看指定数据库中所有自定义函数名
select name from MySQL.proc 
where db='course' and type='function';

--查看指定函数名的详细信息
show create function function_name;

--函数信息保存在information_schema数据库中的routines表中，检索表可查看相关函数信息
select * from information_schema.routines
	where routine_name = 'function_name';
```

修改函数

```sql
格式
alter function func_name[characters -]
```

删除函数定义

```sql
drop function func_name;
```

## 语句

### 控制流语句

if

```sql
--格式（其中“[]”中为可选并非使用时要带上 []）
if condition then
...
[else condition then]
...
[else]
...
```

case

```sql
-- 格式
case value
	when vlaue then
	...
	else
	...
end case
```

### 条件判断函数

if()

```sql
if (condition , v1, v2)
condition为true,返回 v1, 否则返回v2
```

ifnull()

```sql
ifnull(v1, v2)
从v1开始，返回第一个不为null的值
```

case

```sql
case 表达式1
when 值1 = 条件1 then 结果1
when 值2 = 条件2 then 结果2
else 其他值
```

### 循环语句

while

```sql
while condition do
...
end while;
```

loop

```sql
loop
...
end loop
```

repeat

```sql
repeat
...
until condition
end repeat
```

## 存储过程

### 基础

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

### 条件和处理程序定义

定义条件

```sql
declare condition_name condition for condition_type;
-- condition_type:
-- sqlstate [value] sqlstate_value|mysql_error_code
-- condition_name：错误触发条件的名称
-- condition_type:条件类型，MySQL错误代码，ANSI标准错误代码
				--sqlstate_value是长度为5的字符串类型错误代码
				--mysql_error_code数值类型错误代码
```

定义处理程序

```sql
declare handler_type handler for condition_value[] sp_statement
```

参数相关

handler_type: 

- 含义：错误处理类型

- 取值：
    - continue发生错误不处理，执行其他语句
    - exit发生错误退出sql语句的执行

condition_value

- 含义：错误触发条件
- 取值:

> sqlstate [value]
>
> sqlstate_value
>
> condition_name
>
> sqlwarning
>
> not found
>
> sqlexception
>
> mysql_error_code

sqlstate_value

- 含义：长度为5字符串类型错误代码

- 取值：

    | condition_value  | 定义的错误条件名称                            |
    | ---------------- | --------------------------------------------- |
    | sqlwarning       | 匹配所有01开头的sqlstate错误代码              |
    | not found        | 匹配所有02开头的sqlstate错误代码              |
    | sqlexception     | 匹配其它非sqlwarning和not found捕获的错误代码 |
    | mysql_error_code | 数值型错误代码                                |

sp_statement

- 含义：自定义错误处理程序

## 游标

> 流程：声明游标->打开游标->提取数据，处理数据->关闭游标
>
> 意义：处理结果集

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

## 事件

认识事件调度

开启事件调度

```sql
set @@global.event_scheduler = true;

在mysql配置文件my.ini中加入 event_scheduler = 1;
```

查看事件

```sql
show variables like 'event_scheduler'

select @@event_scheduler
```

基础操作

创建

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

管理事件

```sql
show events [from schema_name] [like 'pattern'|where expr]

show events;

show evetns\G;

show create event event_name;
```

修改事件

```sql
alter event event_name [on schedule schedule]
[rename to new_name] on completion[not] preserve]
[comment'comment'][enable|disable][db sql_statement]
```

删除事件

```sql
drop event[if exists][database name.]event_name
```

# 
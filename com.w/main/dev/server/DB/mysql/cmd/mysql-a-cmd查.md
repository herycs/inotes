# mysql-查找

## 单记录

```sql
select * from tb_demo1;

select * from tb_demo1
	where age1 = 26 and new_column = 159;
```

```sql
-- 模糊匹配
	
-- 配合模式匹配的语法查找，即就是like 或者 not like 后面的引号内的内容
-- 若模式匹配串中 _ 一般为通配符，若要当一般字符使用需要转义，即就是：\_
select * from tb_demo1 where name1 like "%L%";
select * from tb_demo1 where name1 not like "%L%";
```

| 匹配符 | 说明                             |
| ------ | -------------------------------- |
| ^      | 匹配文本开始字符                 |
| $      | 匹配文本结束字符                 |
| .      | 匹配任何单个字符                 |
| *      | 匹配0或多个在它前面的字符        |
| +      | 匹配前面的字符1或多次            |
| <>     | 匹配指定的字符串                 |
| []     | 匹配字符集合中的任意一个         |
| {n,}   | 匹配前面的字符串至少n次          |
| {m,n}  | 匹配前面的字符串至少m次，至多n次 |
| [^]    | 匹配不在括号中的任何字符         |

```sql
select * from scc where Sno regexp "21$";

select * from scc where Sno regexp "^20";

select * from scc where Sno regexp ".15122";

select * from scc where Sno regexp "[12151]";

select * from scc where Sno regexp "[^12151]";

select * from scc where Sno regexp "1{2,}";

select * from scc where Sno regexp "2{1,2}";
```

## 聚合记录

distinct 表示取消重复值，

all 或者默认（all） 表示不取消重复值重复

- 除了count(*)外，均跳过空值，只处理非空值

- 使用范围：where字句中不可使用

    ​					只能适用于select 和 group by 中的having 子句

```sql
select count(*) from tb_demo1;
select sum( all age1) from tb_demo1;
select avg(distinct grade2) from tb_demo1;
select Max(grade2) from tb_demo1;
select min(grade2) from tb_demo1;
```



## 联合查询

拼接 + 查询

- 拼接：依据条件，将表拼接到一起

- 查询：基础操作，将所需要的字段值查询出来

> 测试前提
>
> ```sql
> create table a ( id int unique primary key not null auto_increment, age int default 0 );
> create table b ( id int unique primary key not null auto_increment, name char(10) default 'll' ); 
> create table c( id int unique primary key not null auto_increment ,
>                age int default 0, name char(4) default 'null' );
> 
> insert into a(age) values (1), (2), (3), (7), (9);
> insert into b(name) values('ll'), ('mm'), ('ss'), ('ww'), ('xx');
> insert into c(age, name) values (22, 'ww'), (33, 'll'), (23, 'cc'), (24, 'ss');
> ```

### 内连接

```sql
-- 连接查询，mysql默认内连接
select * from a a1, b b1 
	where a1.id = b1.id && a1.id > 3;
-- 内连接
select * from a a1 join b b1 on a1.id = b1.id where a1.age > 3;
```



### 外连接

```sql
-- 左外连接
select * from b b1 left join a a1 on a1.id = b1.id where a1.age > 3;
-- 右外连接
select * from a a1 right join b b1 on a1.id = b1.id where a1.age > 3;
```



### 自然连接

```sql
-- 自然连接
select * from a student natural join b;

-- 子查询
select * from c where id = (select id from b where name = 'll');
select * from c where age > (select id from b where name = 'ww');
```

### 子查询

```sql
select age1 from tb_demo1 where tb_demo1.name1 in (
	select tb_demo2.name2 from tb_demo2 where age2 > 16); 
```

- 带有 any (some) 或 all的谓词子查询

    ```sql
    -- 查询所有非cs系的学生年年龄比cs系学生的年龄小的学生
    select Sname, Sage from student where Sage < any(
    	select Sage from student where Sdept = 'cs'
    ) 
    and Sdept<>'cs';
    ```

- 带有 exists的谓词查询

    ```sql
    -- 选取学号为1的学生，输出姓名
    select Sname from student where exists(
    	select * from sc where Sno = student.Sno and Cno = '1'
    );
    ```

- 带有 not exists 的谓词子查询

    ```sql
    -- 选取学号不为1的学生，输出姓名
    select Sname from student where not exists(
    	select * from sc where Sno = student.Sno and Cno = '1'
    );
    ```

## 结果集处理

```sql
-- 排序
select * from tb_demo1 order by grade2 DESC;
```

### 分组

```sql
-- 分组，选择 grade2和age1列，按grade2分组，并对每组的age1统计有值的
select grade2, count(age1) from tb_demo1 group by grade2;

-- 分组后筛选
select grade2, count(age1) from tb_demo1 group by grade2 having count(age1) > 1;
```

### 合并

```sql
-- 同时满足两方
-- 去重
select * from student where Sdept = 'cs'
union
select * from student where Sage = 19;
-- 不去重
select * from student where Sdept = 'cs'
union all
select * from student where Sage = 19;
```

交: intersect

> sql语法中有except关键字，但是mysql没有

差: except

>  mysql不支持直接的差操作，只能通过其他方式达到目的

基于派生表的查询

## 特殊值处理

```sql
select * from tb_demo1 where age1 > 10 and age1 < 22;
select * from tb_demo1 where age1 != 22;
select * from tb_demo1 where name1 is null;
select * from tb_demo1 where name1 is not null;
select * from tb_demo1 where not age1 < 22;
```


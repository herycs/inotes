# 基本操作

## 库

### 管理

创建

```sql
create database;
create database if not exists school default charset utf8mb4;;
```

删除

```sql
drop database ss;
```

使用库

```sql
use db_name
```

## 表

### 存储引擎表空间

### 临时表管理

### 数据表管理

创建

```sql
create table tb_demo1(
	name1 varchar(12),
    age1 int
);
```

增加

```sql
-- 添加值
insert into tb_demo1(name,age) values('lihua', 12);
```

删除

```sql
-- 删除表
drop table tb_demo1;
```

```sql
-- 删除值
delete from tb_demo1 where naem = "haung";
```

更新

```sql
update tb_demo1 set age1 = age+1;
update tb_demo1 set name1 = "ling" where id = 1923924;
```

修改

修改名称

```sql
-- 修改表名
alter table tb_demo1 rename to t_demo1;
-- 修改列名
alter table tb_demo1 change column grade grade2 varchar(23);
```

修改结构

```sql
-- 修改表列类型
alter table tb_demo1 modify grade varchar(12);
```

```sql
-- 添加列
alter table tb_demo1 add column new_column varchar(12);
-- 删除列
alter table tb_demo1 drop name1;
```

约束修改

```sql
-- 添加主键约束
alter table tb_demo1 add constraint name1 primary key tb_demo1(name1);

-- 添加唯一性约束
-- 唯一约束：可以在一个字段，一组字段或一个表上定义唯一性约束，保证值不相同
alter table tb_demo1 add constraint name1 unique (name1);
alter table tb_demo1 add constraint name1 unique tb_demo2(name2);

-- 添加检查约束(约束值的范围)
alter table sss add constraint age check(age > 0 and age < 40);
```

附加操作

drop column：
-- 设定如下时，对应的效果

- cascade	删除引用该列的的其他对象，表上有视图时可以删除

- restrict	若被引用拒绝删除，有试图时拒绝删除

```sql
-- 示例
drop table tb_demo1 restrict;
drop table tb_demo1 cascade;

-- 用于修改原有的定义
alter column:
-- 用于删除原有的约束条件
drop constraint:
```

## 数据检索

#### 单表查询

> 无差别查找

```sql
select * from tb_demo1;
```

> 精确查找

```sql
select * from tb_demo1 where age1 = 26 and new_column = 159;
```

> 模糊查找

```sql
-- 配合模式匹配的语法查找，即就是like 或者 not like 后面的引号内的内容
-- 若模式匹配串中 _ 一般为通配符，若要当一般字符使用需要转义，即就是：\_
select * from tb_demo1 where name1 like "%L%";
select * from tb_demo1 where name1 not like "%L%";
```

> 范围查找

```sql
select * from tb_demo1 where age1 > 10 and age1 < 22;
select * from tb_demo1 where age1 != 22;
select * from tb_demo1 where name1 is null;
select * from tb_demo1 where name1 is not null;
```

> 多重条件

```sql
select * from tb_demo1 where age1 < 22 and name1 like "%w%";
select * from tb_demo1 where age1 < 22 or name1 like "%w%";
select * from tb_demo1 where not age1 < 22;
```

#### 聚合函数查询

- order by

    ```sql
    select * from tb_demo1 order by grade2 DESC;
    ```

- 聚集函数

    > distinct 表示取消重复值，
    >
    > all 或者默认（all） 表示不取消重复值重复
    >
    > - 除了count(*)外，均跳过空值，只处理非空值
    >
    > - 使用范围：where字句中不可使用
    >
    >     ​					只能适用于select 和 group by 中的having 子句

    ```sql
    select count(*) from tb_demo1;
    select sum( all age1) from tb_demo1;
    select avg(distinct grade2) from tb_demo1;
    select Max(grade2) from tb_demo1;
    select min(grade2) from tb_demo1;
    ```

- group by

    ```sql
    //选择 grade2和age1列，按grade2分组，并对每组的age1统计有值的
    select grade2, count(age1) from tb_demo1 group by grade2;
    ```

- having

    ```sql
    -- 分组后还要筛选
    select grade2, count(age1) from tb_demo1 group by grade2 having count(age1) > 1;
    ```

#### 连接查询

- 等值非等值连接查询

    > 字段类型必须可比

    ```sql
    select tb_demo1.*, tb_demo2.* from tb_demo1, tb_demo2 where tb_demo1.name1 = tb_demo2.name2;
    ```

- 循环嵌套连接

- 自然连接

- 外连接

    ```sql
    select name1, age1, new_column, grade2, tb_demo2.name2 ,tb_demo2.grade3 from tb_demo1, tb_demo2 where tb_demo1.name1 = tb_demo2.name2;
    ```

- 多表连接

    > 一个表与自身连接,两个表进行连接

#### 嵌套查询

- 带有in 的谓词查询

    ```sql
    select age1 from tb_demo1 where tb_demo1.name1 in (
    	select tb_demo2.name2 from tb_demo2 where age2 > 16); 
    ```

- 带有比较运算符的子查询

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

#### 集合查询

并: union

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

- 基于派生表的查询

    > 子查询出现在from语句中


#### 正则表达式模糊查询

> 语法格式：select...from where ... regexp "";

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

- 示例

    ```sql
    select * from scc where Sno regexp "21$";
    
    select * from scc where Sno regexp "^20";
    
    select * from scc where Sno regexp ".15122";
    
    select * from scc where Sno regexp "[12151]";
    
    select * from scc where Sno regexp "[^12151]";
    
    select * from scc where Sno regexp "1{2,}";
    
    select * from scc where Sno regexp "2{1,2}";
    ```


## 索引

创建

基本语法：create <kbd>demo</kbd>index <kbd>name</kbd>on<kbd>tb_demo1</kbd>(<kbd>column_1 </kbd> &nbsp; <kbd>sort</kbd>_, <kbd>column_2  </kbd>&nbsp;<kbd>sort</kbd>_);

> demo取值两个：
>
> - unique:此索引的每个值只对应唯一的一个数据记录
>
> - cluster:要建立的索引是聚簇索引
>
> name_:创建的索引的名字
>
> tb_demo1:表名
>
> column_1,column_2为列名
>
> sort取值: ASC升序，DESC降序，默认ASC

修改

```sql
alter index <old_name> rename to <new_name>;
```

删除

```sql
-- name_ 为索引名
drop index name_;
```

## 视图

> #### 	视图是加了限制的sql操作,但其实质还是作用于数据库本身数据的
>
> #### 	目的：对操作的用户隐藏他不需要看见的数据

创建

> 创建视图时属性要不全选或者全不选,没第三个选择
>
> 以下情况需要指定所有列名:
>
> - 某个列是聚集函数或者列表达式
> - 多表连接时有重名的字段
> - 须在视图中修改某个列名

- 单表视图

```sql
-- 本视图仅对一个表创建，省略视图后的属性名可以
create view IS_student
as
select Sno, Sname, Sage from student where Sdept = 'cs';
```

- 多表视图

```sql
-- 本视图针对多个表创建，不可省略
create view IS_SI(Sno, Sname)
as
select Student.Sno, Sname from student, sc where Sdept = 'IS' and student.Sno = sc.Sno and sc.sno = '1';
```

- 添加使用条件的视图

```sql
-- 本例,中with check option指定后续使用此视图的操作需要满足视图中定义的谓词条件
create view IS_student
as
select Sno, Sname, Sage from student where Sdept = 'cs'
with check option;
```

删除

```sql
-- 级联删除
-- 将删除视图和由此视图导出的所有视图
drop view IS_SI cascade;
```

查询

> 视图查询时
>
> - 有效性检查,视图存在则执行下列操作
> - 从数据字典中取出视图
> - 将定义中的子查询和用户的查询结合起来,转换为等价的对基本表的查询------(视图消解)
> - 执行修正了的查询

```sql
-- 查询视图操作
select Sno, Sage from IS_student where Sage < 20;
-- 转换后的语句(视图消解后的语句)
select Sno, Sage from student where Sdept='IS' and Sage < 20;
```

修改

```sql
update IS_student set Sname = '233' where Sno = '201215122';
```

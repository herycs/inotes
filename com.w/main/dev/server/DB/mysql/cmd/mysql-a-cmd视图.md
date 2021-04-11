# 视图

> #### 	视图是加了限制的sql操作,但其实质还是作用于数据库本身数据的

## 创建

> 创建视图时属性要不全选或者全不选,没第三个选择
>
> 以下情况需要指定所有列名:
>
> - 某个列是聚集函数或者列表达式
> - 多表连接时有重名的字段
> - 须在视图中修改某个列名

```sql
-- 本视图仅对一个表创建，省略视图后的属性名可以
create view IS_student
as
select Sno, Sname, Sage from student where Sdept = 'cs';

-- 本视图针对多个表创建，不可省略
create view IS_SI(Sno, Sname)
as
select Student.Sno, Sname from student, sc where Sdept = 'IS' and student.Sno = sc.Sno and sc.sno = '1';

-- 本例,中with check option指定后续使用此视图的操作需要满足视图中定义的谓词条件
create view IS_student
as
select Sno, Sname, Sage from student where Sdept = 'cs'
with check option;
```

## 删除

```sql
-- 级联删除
-- 将删除视图和由此视图导出的所有视图
drop view IS_SI cascade;
```

## 查询

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

## 修改

```sql
update IS_student set Sname = '233' where Sno = '201215122';
```


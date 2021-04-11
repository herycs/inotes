# mysql-删除

## 数据库

```sql
drop database ss;
```

## 表

```sql
  -- 删除表
  drop table tb_demo1;
```

## 数据

```sql
  -- 删除指定数据,并记录日志
  delete from tb_demo1 where naem = "haung";
  -- 清理全部数据，相关数据重置为基础值，但不记录日志
  truncate table tb_demo1;
```

## 结构

```sql  -- 删除列
  alter table tb_demo1 drop name1;
```

> drop column：
> -- 设定如下时，对应的效果
>
> - cascade		删除引用该列的的其他对象，表上有视图时可以删除
>
> - restrict	若被引用拒绝删除，有视图时拒绝删除
>
> ```sql
> -- 示例
> drop table tb_demo1 restrict;
> drop table tb_demo1 cascade;
> ```
>
> alter column:
> -- 用于修改原有的定义
>
> drop constraint:
> -- 用于删除原有的约束条件

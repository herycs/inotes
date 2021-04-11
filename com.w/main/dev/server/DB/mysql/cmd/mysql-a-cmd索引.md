# mysql-索引

## 创建

基本语法：create <kbd>demo</kbd>index <kbd>name</kbd>on <kbd>name</kbd> <kbd>tb_demo1</kbd>(<kbd>column_1 </kbd> &nbsp; <kbd>sort</kbd>_, <kbd>column_2  </kbd>&nbsp;<kbd>sort</kbd>_);

> 其中：
>
> demo取值两个：
>
> - unique:此索引的值只对应唯一的一个数据记录
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

## 修改

```sql
alter index <old_name> rename to <new_name>;
```

## 删除

```sql
-- name_ 为索引名
drop index name_;
```

## 查询

```mysql
show index from {table_name};
```

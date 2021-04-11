# mysql-修改

## 表

```sql
  -- 修改表名
  alter table tb_demo1 rename to t_demo1;
```

## 数据

```sql
  update tb_demo1 set age1 = age+1;
  update tb_demo1 set name1 = "ling" where id = 1923924;
```

## 结构

```sql
  -- 修改列名
  alter table tb_demo1 change column grade grade2 varchar(23);
  -- 修改表列类型
  alter table tb_demo1 modify grade varchar(12);
```


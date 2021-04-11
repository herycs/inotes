# mysql-增加

## 数据库

```sql
create database ss;
```

## 表

```sql
create table tb_demo1(
	name1 varchar(12),
    age1 int
);
create table tb_demo2(
  	name2 varchar(13),
    age2 int
  );
```

## 数据

```sql
insert into tb_demo1(name,age) values('lihua', 12);
```

## 结构

```sql
-- 添加列
alter table tb_demo1 add column new_column varchar(12);

-- 添加主键约束
alter table tb_demo1 add constraint name1 primary key tb_demo1(name1);
alter table tb_demo1 add constraint name1 primary key tb_demo2(name2);

-- 添加唯一性约束
-- 唯一约束：可以在一个字段，一组字段或一个表上定义唯一性约束，保证值不相同
alter table tb_demo1 add constraint name1 unique (name1);
alter table tb_demo1 add constraint name1 unique tb_demo1(name1);
alter table tb_demo1 add constraint name1 unique tb_demo2(name2);

-- 添加检查约束(约束值的范围)
alter table sss add constraint age check(age > 0 and age < 40);
```


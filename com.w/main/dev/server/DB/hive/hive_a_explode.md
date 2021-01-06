# hive 炸开数据

## hive 中支持结构体存储数据

但在获取数据子项时，难免会存在嵌套层级较深的情况

这时需要将字段取出，解析一层一层数据的过程就像“炸开”一样，故也称 explode()为爆炸函数

## 使用示例

仿写SQL

```sql
select
	name,
	age,
	mp.id as id,
	mp.value as value
from tb1
	lateral view explode(data.mark.items.part) part as mp
limit 30
```

## 应用效果

### 数组

对于array数组，炸开数据，例如：id = 3, value = [1,2,3]

```sql
select
	id,
	vi as value
from tb1
	lateral view explode(data.mark.items.value) value as vi
```

效果

```sql
id		value
3		1
3		2
3		3
```

### 结构体

对于struct数据，想要取到内部数据 则需要使用 array() 转换结构体为数组类型

```sql
select
	id,
	v as value
from tb1
	lateral view explode(array(data.mark.items.value)) value as v
```

效果

```sql
id		value
1		[1,2,3]
```

若需要取结构体内部数组，则只需要 使用 field1.field2.xxx.xxx的格式，从顶级字段一路指向该数组即可

```sql
select
	id,
	v as value
from tb1
	lateral view explode(data.mark.items.value) value as v
```

效果

```sql
id		value
1		[1,2,3]
```






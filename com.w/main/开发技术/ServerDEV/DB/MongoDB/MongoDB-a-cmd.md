# MongoDB-CMD

## CRUD操作

查询

```shell
db.collection.find(
	{age: {$lt: 10}}
)
```

增加

- insertOne()
- insertMany()

```shell
db.collection.insertOne(
    {
      name: "lili",
      age: 12,
      status: "run"
    }
)
```

修改

- updateOne()
- updateMany()
- replaceOne()

```shell
db.collection.updateMany(
	{age: {$lt: 5}},
	{$set: {status: "version 1.2"}}
)
```

删除

- deleteOne()
- deleteMany()

```shell
db.collection.deleteOne(
	{age : 6}
)
```

## 文字搜索

> 1. 建立索引
> 2. 进行查询

创建索引

```shell
db.collection.createIndex(
	{
	  name: "text",
	  status: "text"
	}
)
```

查询

```shell
# 普通匹配：包含 dev | run
db.collection.find(
	{
		$text: {$search: "dev run"}
	}
)
# 精确匹配:包含dev run
db.collection.find(
	{
		$text: {$search: "\"dev run\""}
	}
)
# 术语排除：包含dev但没有run
db.collection.find(
	{ $text: {$search: "dev -run"} }
)
```

## 聚合


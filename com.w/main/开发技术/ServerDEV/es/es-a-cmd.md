# ES-cmd

> 操作数据：put类型操作数据均以json格式传输
>
> CRUD操作类型识别基于RESTFUL风格进行

## 集群监控

集群状态

```shell
curl -XGET localhost:9200/_cat/health?v
```

集群节点列表

```shell
curl -XGET localhost:9200/_cat/nodes?v
```

## 索引管理

> 请求路径直接操作域名后即可

查看所有

```shell
curl -XGET localhost:9200/_cat/indices?v
```

创建索引

```shell
-- put 类型请求
curl -XPUT localhost:9200/xxx
```

查询索引

```shell
curl -XPUT localhost:9200/xxx/yyy/1
```

删除

```shell
curl -XDELETE localhost:9200/customer?xxx
```

## 数据管理

> 操作域名及端口后需要写上相关参数

查找

```shell
curl -XPUT localhost:9200/itaobao_es/article/_search
```

增加

```shell
curl -XPUT localhost:9200/itaobao_es/article/1
```

删除

```shell
curl -XDELETE localhost:9200/customer/external/2
```

修改

```shell
curl -XPUT localhost:9200/itaobao_es/article/_update
```

批处理

```shell
curl -XPOST localhost:9200/customer/external/_bulk
```


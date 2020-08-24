# 缓存过期策略

## 配置内存

- redis.conf中配置：maxmemory 100mb
- 命令：config set maxmemory 100mc

若不设置最大内存或者最大为0，64位无限制，32位下3G

## 内存淘汰策略

noeviction(默认)：写请求直接返回错误

allkeys-lru：LRU处理所有key

volatile-lru：设置过期时间的key中使用LRU，没有可被淘汰的key返回错误

allkeys-random：所有key中随机淘汰数据

volatile-random：设置过期时间的key中随机淘汰，没有可被淘汰的key返回错误

volatile-ttl：设置过期时间的key中，依据其过期时间（越早优先被淘汰），没有可被淘汰的key返回错误

### 配置淘汰策略

```shell
config get maxmemory-policy
// 配置文件
maxmemory-policy allkeys-lru
config set maxmemory-policy allkeys-lru
```

### java实现lru

- 获取值则将其移动到尾部
- 删除值将其直接删除

### Redis中的近似LRU

每次随机出5个(这个采样数据可以设定)key从中淘汰一个key

数据结构，每个key增加24bit保存最后一次访问的时间

### Redis3.0中的LRU

优化Redis3.0维护一个候选池（16），池中数据依据访问时间进行排序，第一次选取的key放入池中，每次随机的key时间少于池中最小时间才放入池中，当放满后，池中最后访问时间的最大的移除

### LFU算法

least frequently used

依据key的最近访问频率淘汰，很少被访问的优先被淘汰，最多被访问的留下来

### LFU

- volatile-lfu：设置了过期时间key中使用LFU算法淘汰
- allkeys-lfu：在所有key中使用LFU算法淘汰数据


# Session

## 处理策略

1.粘性session

粘性session是指Ngnix每次都将同一用户的所有请求转发至同一台服务器上，即将用户与服务器绑定。

2.服务器session复制

即每次session发生变化时，创建或者修改，就广播给所有集群中的服务器，使所有的服务器上的session相同。

3.session共享

缓存session，使用redis， memcached。

4.session持久化

将session存储至数据库中，像操作数据一样才做session。

### 粘性Session

优点：实现简单，对应用无侵入性，无额外开销。

缺点：一旦某个web服务器重启或宕机，相对应的session的数据将会丢失，且其对nginx负载均衡的能力进行了弱化。

### Session复制

优点：对应用无侵入性，重启或宕机不会造成session的丢失。

缺点：需要依赖支持session复制的web服务器，在数据量很大的情况下不仅占用网络资源，而且会导致延迟。只适用于web服务器比较少且session数据量少的情况。

这种方式在应用集群达到数千台的时候，就会出现瓶颈，每台都需要备份session，出现内存不够用的情况。

### Session共享

session服务器，利用独立部署的session服务器（集群）统一管理session，服务器每次读写session时，都访问session服务器

优点：可靠性高，减少了web服务器的资源消耗，服务器重启或宕机不会造成session的丢失。

缺点：实现上有些复杂，配置较多，增加了一次从存储介质中获取session数据的开销。适用于web服务器较多，要求高可用的情况。

### Session持久化
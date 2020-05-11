## Java操作Redis

## Jedis

### 使用步骤

1. 下载jedis的jar包

2. 使用

    ```java
    //1. 获取连接
    Jedis jedis = new Jedis("localhost",6379);
    //2. 操作
    jedis.set("username","zhangsan");
    //3. 关闭连接
    jedis.close();
    ```

### Jedis操作各种redis中的数据结构

#### 字符串类型 string

> set
>
> get

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

//2. 操作
//存储
jedis.set("username","liming");
//获取
String username = jedis.get("username");

//可以使用setex()方法存储可以指定过期时间的 key value
jedis.setex("activecode",20,"item");//将activecode：hehe键值对存入redis，并且20秒后自动删除该键值对

//3. 关闭连接
jedis.close();
```

#### 哈希类型 hash ： map格式  

> hset
> hget
> hgetAll

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

//2. 操作
// 存储hash
jedis.hset("user","name","lili");
jedis.hset("user","age","23");
jedis.hset("user","gender","123");

// 获取hash
String name = jedis.hget("user", "name");

// 获取hash的所有map中的数据
Map<String, String> user = jedis.hgetAll("user");

// keyset
Set<String> keySet = user.keySet();
for (String key : keySet) {
    //获取value
    String value = user.get(key);
    System.out.println(key + ":" + value);
}

//3. 关闭连接
jedis.close();
```



#### 列表类型 list ： linkedlist格式。支持重复元素

> lpush / rpush
> lpop / rpop
> lrange start end : 范围获取

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

//2. 操作
// list 存储  
jedis.lpush("mylist","a","b","c");//从左边存
jedis.rpush("mylist","a","b","c");//从右边存

// list 范围获取
List<String> list = jedis.lrange("mylist", 0, -1);
System.out.println(list);

// list 弹出
String element = jedis.lpop("list");//c
System.out.println(element);

String element1 = jedis.rpop("list");//c
System.out.println(element1);

// list 范围获取
List<String> list2 = jedis.lrange("list", 0, -1);
System.out.println(list2);

//3. 关闭连接
jedis.close();
```



#### 集合类型 set  ： 不允许重复元素

> sadd
> smembers:获取所有元素

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

//2. 操作
// set 存储
jedis.sadd("my","java","php","c++");

// set 获取
Set<String> set = jedis.smembers("my");
System.out.println(set);

//3. 关闭连接
jedis.close();
```

#### 有序集合类型 sortedset：不允许重复元素，且元素有顺序

> zadd
> zrange

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

//2. 操作
// sortedset 存储
jedis.zadd("mysortedset",3,"test");
jedis.zadd("mysortedset",30,"info");
jedis.zadd("mysortedset",55,"hex");

// sortedset 获取
Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);

//3. 关闭连接
jedis.close();
```

## jedis连接池： JedisPool

### 使用

1. 创建JedisPool连接池对象
2. 调用方法 getResource()方法获取Jedis连接

代码

```java
//0.创建一个配置对象
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(50);
config.setMaxIdle(10);

//1.创建Jedis连接池对象
JedisPool jedisPool = new JedisPool(config,"localhost",6379);

//2.获取连接
Jedis jedis = jedisPool.getResource();
//3. 使用
jedis.set("hehe","heihei");

//4. 关闭 归还到连接池中
jedis.close();
```

​		

### 连接池工具类

```java
public class JedisPoolUtils {

    private static JedisPool jedisPool;

    static{
        //读取配置文件
        InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
        //创建Properties对象
        Properties pro = new Properties();
        //关联文件
        try {
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //获取数据，设置到JedisPoolConfig中
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));

        //初始化JedisPool
        jedisPool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));

        //获取连接方法
        public static Jedis getJedis(){
            return jedisPool.getResource();
        }
    }
```



## 注意

使用redis缓存一些不经常发生变化的数据。

* 数据库的数据一旦发生改变，则需要更新缓存。
    * 数据库的表执行 增删改的相关操作，需要将redis缓存数据情况，再次存入
    * 在service对应的增删改方法中，将redis数据删除。


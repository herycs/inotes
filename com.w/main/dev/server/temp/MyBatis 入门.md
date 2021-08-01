# MyBatis 入门

> ORM  - 映射框架
>
> 完成：程序内部数据结构（对象）------ MySQL表数据 之间的映射关系

## 1.介绍

### 1.1基本介绍

*MyBatis* 灵活的持久层框架，它支持自定义SQL、存储过程以及高级映射。

### 1.2同类型产品对比

Hibernate 

本身基于全量的数据映射，Spring Data JPA基于此种策略完成，数据映射

- 优点：

开发简单：大多数操作及SQL已由底层进行了封装，开箱即用

适配：对不同的数据库类型可以无缝切换

缓存：（session缓存，二级缓存，查询缓存）

- 缺点：

SQL灵活度低

不支持批处理机制：如果要批量更新或插入数据时，还需要显示的flush，clear之类的操作，操作复杂

调试不易：封装的东西过多，若设置不到位可能会比较难发现错误

### 1.3市场占比

MyBatis 简单易上手适用于互联网公司

Hinerbate 适用于企业级开发

### 1.4应用

Spring Data JAP提供了对DB操作的封装，相对而言编写简单

Hibernate 的调优配置方面，需要考虑的较多

Mybatis 入门简单，配置灵活，更多作为开发者学习的技术

## 2.基本信息介绍

### 2.1基本标签

```sql
select update insert delete

resultMap

if foreach choose

where trim set

collection association

sql include
```

select

#### 属性

**id**  唯一的名称，对应dao中mapper的接口名称

**paramterType**   定义传入的参数类型

**resultType**   返回数据类型对应实体类

**resultMap**  外部 resultMap 的命名引用

**flushCache** 将其设置为 true，语句被调用会导致本地缓存和二级缓存都会被清空，默认值：false

**useCache** 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：true

**timeout**  这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。

默认值为 unset（依赖驱动）,没有返回结果就抛出异常

**fetchSize** 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）,比如设置为20，而你要查询的总记录是100条，驱动会每次批量查询20条，共查询5次，myabtis会将这5次结果聚合一起返回100条。注意Mysql是不支持fetchSize的

**statementType** STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED

**resultSetType** FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）

**databaseId** 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。

**resultOrdered** 这个设置仅针对嵌套结果 select 语句适用：如果为true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至导致内存不够用。默认值：false。

**resultSets** 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。

foreach

item，index，collection，open，separator，close

**collection** 要做foreach的对象

作为入参时，List对象默认用"list"代替作为键，数组对象有"array"代替作为键，Map对象没有默认的键。

当然在作为入参时可以使用@Param("keyName")来设置键，设置keyName后，list,array将会失效。 

除了入参这种情况外，还有一种作为参数对象的某个字段的时候。

**item** 合中元素迭代时的别名，必选

**index** 在list和数组中,index是元素的序号，在map中，index是元素的key，可选

**open** foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。可选

**separator** 元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样，可选。

**close** foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。

#### 使用

```xml
<if test="item != null and item != ''"></if>

别名
<typeAliases>
  <typeAlias type="com.gqlofe.userinfo.bean.entity.UserInfo" alias="User"/>
</typeAliases>

prefix 在trim标签内容前面加上前缀
suffix 在trim标签内容后面加上后缀
prefixOverrides 移除前缀内容。即 属性会忽略通过管道分隔的文本序列，多个忽略序列用 “|” 隔开。它带来的结果就是所有在 prefixOverrides 属性中指定的内容将被移除。
suffixOverrides 移除前缀内容。
<trim prefix="(" suffix=")" suffixOverrides=",">

  <where>
    <choose>
      <when test="id != null and test.trim() != '' ">
        id = #{id}
      </when> 
      <when test="name != null and name.trim() != '' ">
        name = #{name}
      </when> 
      <otherwise>
        age = 17  
      </otherwise>
    </choose>
  </where>
  
  <resultMap id="BaseResultMap" type="com.gqlofe.userinfo.bean.entity.UserInfo"></resultMap>
```

### 2.2运行流程

1. **注册驱动**
2. **获取Connection连接**
3. **执行预编译**
4. **执行SQL**
5. **封装结果集**
6. **释放资源**

1、读取MyBatis的核心配置文件。mybatis-config.xml为MyBatis的全局配置文件，最终会被封装成一个Configuration对象。

用于配置数据库连接、属性、类型别名、类型处理器、插件、环境配置、映射器（mapper.xml）等信息

2、加载映射文件。映射文件即SQL映射文件，该文件中配置了操作数据库的SQL语句，映射文件是在mybatis-config.xml中加载；可以加载多个映射文件。常见的配置的方式有两种，一种是package扫描包，一种是mapper找到配置文件的位置。

```java
<!-- 使用包路径，扫描包下所有的接口，这种方式比较方便 --> 

<package name="com.mybatis.demo"/> 
<!-- resource:使用相对路径的资源引用-->
<!-- url:使用绝对类路径的资源引用-->
<!-- class:使用映射器接口实现类的完全限定类名-->
<mapper resource="xxx.xml"/>
```

3、构造会话工厂获取SqlSessionFactory。

这个过程其实是用建造者设计模式使用SqlSessionFactoryBuilder对象构建的，SqlSessionFactory的最佳作用域是应用作用域。

```text
//2. 创建SqlSessionFactory对象实际创建的是DefaultSqlSessionFactory对象
SqlSessionFactory builder = new	SqlSessionFactoryBuilder().build(inputStream);
```

4、创建会话对象SqlSession。

由会话工厂创建SqlSession对象，对象中包含了执行SQL语句的所有方法，线程私有。

SqlSession的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。

```text
//3. 创建SqlSession对象实际创建的是DefaultSqlSession对象
  SqlSession sqlSession = builder.openSession();
```

5、Executor执行器。

负责SQL语句的生成和查询缓存的维护，它将根据SqlSession传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护

- SimpleExecutor -- SIMPLE 就是普通的执行器。
- ReuseExecutor-执行器会重用预处理语句（PreparedStatements）
- BatchExecutor --它是批处理执行器

6、MappedStatement对象。

MappedStatement是对解析的SQL的语句封装，一个MappedStatement代表了一个sql语句标签，如下：

```text
<!--一个动态sql标签就是一个`MappedStatement`对象-->
<select	id="selectUserList"	resultType="com.mybatis.User"> 
  select * from t_user
</select>
```

7、输入参数映射。

输入参数类型可以是基本数据类型，也可以是Map、List、POJO类型复杂数据类型，这个过程类似于JDBC的预编译处理参数的过程，有两个属性 parameterType和parameterMap

8、封装结果集。

可以封装成多种类型可以是基本数据类型，也可以是Map、List、POJO类型复杂数据类型。

封装结果集的过程就和JDBC封装结果集是一样的。也有两个常用的属性resultType和resultMap。

### 2.3运行时结构

SqlSession

Executor

StatementHandler(ParameterHandler & TypeHandler & ResultSetHandler)

[Mybatis标签详解](https://www.jianshu.com/p/a00670c8f62b)
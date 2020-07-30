# Mybatis

## 地位

ORM框架，负责处理对数据库的连接，提供数据转换以及查询优化等操作

## 组成

配置文件-》sqlSessionFactory-》产生SQLSession-负责处理Application的请求（DefaultSqlSessionFactory），Executor负责执行具体查询操作

### 缓存

SqlSession一级缓存

- 一级缓存是基于 HashMap 的本地缓存
- SqlSession私有

SqlSession 执行 insert、update、delete 操做并提交到数据库时，会清空缓存，保证缓存中的信息是最新的。

mapper二级缓存

- 基于 HashMap 进行存储
- SqlSession共享
- 作用域：mapper 的同一个 namespace

## 动态SQL

1. if 语句 

    简单的条件判断

2. choose 

    (when,otherwize) ,相当于java 语言中的 switch ,与 jstl 中的choose 很类似.

3. trim

    对包含的内容加上 prefix,或者 suffix 等，前缀，后缀

4. where

    主要是用来简化sql语句中where条件判断的，能智能的处理 and or ,不必担心多余导致语法错误

5. set 

    主要用于更新时

6. foreach 

    在实现 mybatis in 语句查询时特别有用


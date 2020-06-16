# Cannot create PoolableConnectionFactory (Access denied for user 'admin'@'localhost' to database 'springtest')

# 1.报错提示：

Request processing failed; nested exception is org.springframework.transaction.CannotCreateTransactionException: Could not open JDBC Connection for transaction; nested exception is java.sql.SQLException: Cannot create PoolableConnectionFactory (Access denied for user <kbd>'admin'@'localhost'</kbd> to database 'springtest')

# 2.注意：

## 2.1常规：报错中admin确实是数据库用户名

> 这种错一般是由于自己的配置方面的问题导致的,建议检查一下几点

## 2.2非常规

- ## 报错中提到的<kbd>'admin'@'localhost'</kbd>中的admin是电脑账号名而不是数据库分配的用户名

# 3.解决方法

- 对于2.1

  - 若使用的是配置文件编写的，即就是：XX.properties文件

    其中的数据例如：user=root中root后不能由空格，其它雷同

  - 检查自己的jar包，注意控制台报错信息

  - 检查配置文件确定，XML文件中的部分数据确实正确，或者能跳转到properties文件中

- 对于2.2

  - 我的原因是配置文件中的,数据库用户名使用的是username,改为user后问题解决
# 有关mysql数据库数据库导出成文件的有关操作和问题

## 1.dump导出文件数据库文件报警

> 在执行以下命令前，应使用 cmd 进入 mysql 的 mysql server 的 bin 命令行下
>
> - 具体操作分两种:
>
>   1.知道mysql server 的位置： 直接 cd 进入该位置
>
>   2.不知道mysql server 的位置：若有mysql的命令行快捷方式可，右键打开文件位置

- 命令：mysqldump -h 地址 -P 端口 -u 用户名 -p 密码  数据库名 > 导出后保存到的地址及文件名

- 示例

  ```mysql
  
  -- <---
  -- 命令1
  -- 简单使用ｄｕｍｐ
  -- 以下语句可以可导出文件（若你的用户所拥有的权限够的话），但会有警告，有关解释请看下面的mysqldump命令，使用警告
  -- -->
  
  mysqldump -h 192.168.1.1 -P 3306 -u root -p 123456 db1 > d:\db1.sql
  
  -- <--
  -- 命令2
  -- 文件导出操作确实可以完成，但针对以上的警告我们可以修改下操作，避免报警
  -- -->
  
  mysqldump -h 192.168.1.1 -P 3306 -u root -p db1 > d:\db1.sql
  
  -- 以上语句执行后，会提示输入password此时输入密码，回车，文件导出完成，且不报警告
  -- 命令不输入密码，而在执行命令后，命令行给出提示后输入密码
  
  ```

- [mysqldump给出Warning: Using a password on the command line interface can be insecure.](https://www.cnblogs.com/fyy-hhzzj/p/8195885.html)

- [官网给出的Using a password on the command line interface can be insecure.解释](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_password)
- [更多sql导出和其它基本操作](https://blog.csdn.net/mengzuchao/article/details/81674460)

- [my.ini文件参数中文注释](https://blog.csdn.net/lienfeng6/article/details/78140404)

- [mysqldump拒绝访问（建议先按照上面的命令尝试一下，可能是命令的不完整）](https://blog.csdn.net/lienfeng6/article/details/78140404)
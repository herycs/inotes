# 模式

## 模式

权限：

获取数据库管理员权限的用户或者拥有管理员赋予的创建模式的权限的用户能创建模式

创建语法格式

> modeName为模式名(不写默认为用户名), username指定该模式创建的对象

```sql
create schema "modeName" authorization username;
```

本质：创建了一个命名空间


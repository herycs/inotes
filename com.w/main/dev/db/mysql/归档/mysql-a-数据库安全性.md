# 安全性(safe)

## 安全性等级

> TCSEC/TDI 分4组7个等级

- A(A1)
- B(B1,B2,B3)
- C(C1,C2)
- D

## 权限

### 授权

- grant

    > 不允许循环授权

    ```sql
    //没有授权资格
    grant select on table student to UI;
    //拥有授权资格
    grant select on table student to UI with grant option;
    ```

### 回收权限

- revoke

    ```sql
    revoke update on table student from UI;
    revoke update on table student from UI cascade;
    revoke update on table student from UI restrict;
    ```


## 模式

> 权限：获取数据库管理员权限的用户或者拥有管理员赋予的创建模式的权限的用户能创建模式

创建

> modeName为模式名(不写默认为用户名), username指定该模式创建的对象

```sql
create schema "modeName" authorization username;
```

> 本质:创建了一个命名空间
>
> 另外create user非sql标准，不同数据库操作可能有所不同

| 拥有的权限 | create user | create schema | create table | 登陆数据库<br />执行数据查询和操作<br /> |
| ---------- | :---------: | :-----------: | :----------: | :--------------------------------------: |
| DBA        |      O      |       O       |      O       |                    O                     |
| RESOURCE   |      X      |       X       |      O       |                    O                     |
| CONNECT    |      X      |       X       |      X       |         O（但必须拥有响应权限）          |

```sql
create user username with DBA;
create user username with resource;
create user username with connect;
```

## 角色

> 被命名的一组与数据库操作相关的权限

创建角色

```sql
create ROLE P1;
```

角色授权

```sql
grant update on table student to name;
```

角色授于用户

```sql
grant P1 to User1;
grant P1 to User1 with admin option;
```

角色权限回收

```sql
revoke update on table student from P1;
```

## 强制存取控制

> 主体：用户/用户进程
>
> 客体：数据文件

敏感度标记

- 对主体而言：许可证级别

- 对客体而言：密级

- 敏感度标记级别

    | 级别名称               | 绝密 | 机密 | 可信 | 公开 |
    | ---------------------- | ---- | ---- | ---- | ---- |
    | 级别大小（用数字代替） | 4    | 3    | 2    | 1    |

- 进入系统时要求遵循下列规则

    - 读取响应客体：主体许可证级别<kbd>>=</kbd>客体密级别
    - 对相应的客体进行写操作：主体许可证级别<kbd><=</kbd>客体级别


认识

- #### 强存取控制对数据本身进行密级标记，无论如何复制，标记与整体是一个不可分的整体

## 审计

- 将用户操作记录入审计日志

- 审计事件

    | 事件名称 |                服务器事件                |                         系统权限                         |    语句事件     |           模式对象事件            |
    | -------- | :--------------------------------------: | :------------------------------------------------------: | :-------------: | :-------------------------------: |
    | 具体     | 数据库服务器的启动，停止，配置文件的加载 | 对所有的结构或模式进行操作上的审计，要求操作符合权限设定 | 对SQL语句而言的 | 对特定模式进行的select或DML的操作 |

- 审计功能

    - audit 设置审计功能
    - noaudit 取消审计功能


## 数据加密

存储加密

|                | 加密方式                         |
| -------------- | -------------------------------- |
| 透明存储加密   | 内核级别加密方式，对用户完全透明 |
| 非透明存储加密 | 多个加密函数实现                 |

传输加密

链路加密

- 传输信息：报头（路由选择信息）和报文（传输信息）
    - 加密方面：报文，报头
    - 安全性较高

端对端加密

- 传输信息：发送端加密，接收端解密
    - 加密方面：只加密报文，不加密报头
    - 安全性较低

加密流程

- 确认双方端点的可靠性
- 协商加密算法和密钥
- 可信数据传输（会话密钥，生命周期仅限于本次通信，理论上每次通信使用的密钥将不同）
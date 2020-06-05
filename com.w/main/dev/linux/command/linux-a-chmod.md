# linux-a-chmod

## 用户管理

> etc/passwd

### 创建

useradd [choise] [username]

choise：

- -c comment 指定一段注释性描述。
- -d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
- -g 用户组 指定用户所属的用户组。
- -G 用户组，用户组 指定用户所属的附加组。
- -s Shell文件 指定用户的登录Shell。
- -u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。

### 删除

userdel -r [username]

### 修改

usermod [choise] [username]

常用的选项包括`-c, -d, -m, -g, -G, -s, -u以及-o等`，这些选项的意义与`useradd`命令中的选项一样，可以为用户指定新的资源值。

### 口令管理

passwd [choise] [username]

- -l 锁定口令，即禁用账号。
- -u 口令解锁。
- -d 使账号无口令。
- -f 强迫用户下次登录时修改口令。

## 用户组

> /etc/group文件的操作

### 添加

groupadd [choise] [group]

- -g GID 指定新用户组的组标识号（GID）。
- -o 一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。

### 删除

groupdel [choist] [group]

- -g GID 为用户组指定新的组标识号。
- -o 与-g选项同时使用，用户组的新GID可以与系统已有用户组的GID相同。
- -n新用户组 将用户组的名字改为新名字

### 切换用户组

newgrp [group]

## 批量添加用户

> root身份执行

编辑用户信息文件：user.txt

```shell
格式：
user001::600:100:user:/home/user001:/bin/bash
```

编辑用户密码文件：passwd.txt

```shell
格式：
user001:123456
```

批量添加用户：newusers < user.txt

编译密码：chpasswd < passwd.txt

密码加密：pwconv
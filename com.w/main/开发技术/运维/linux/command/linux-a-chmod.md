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

## 权限管理

### 基础

linux中文件的操作权限及类型采用十位字符表示，这十位分别为：\- r w x r w x r w x

- 第一位：文件类型

    > d代表的是目录(directroy)
    > -代表的是文件(regular file)
    > s代表的是套字文件(socket)
    > p代表的管道文件(pipe)或命名管道文件(named pipe)
    > l代表的是符号链接文件(symbolic link)
    > b代表的是该文件是面向块的设备文件(block-oriented device file)
    > c代表的是该文件是面向字符的设备文件(charcter-oriented device file)

- 第一个三位：文件所有者权限

- 第二个三位：文件所有者同组成员权限

- 第三个三位：其他用户权限

若采用十二位的表示法，十二位分别为：十位分别为：S T G r w x r w x r w x

- STG：分别代表SUID权限（4）、SGID权限（2）、和 粘滞位权限（1），其他同十位
    - suid/sgid是为了使“没有取得特权用户要完成一项必须要有特权才可以执行的任务”而产生的。 一般用于给可执行的程序或脚本文件进行设置，其中SUID表示对属主用户增加SET位权限，SGID表示对属组内用户增加SET位权限。执行文件被设置了SUID、SGID权限后，任何用户执行该文件时，将获得该文件属主、属组账号对应的身份。（来自[博客](https://blog.csdn.net/u013197629/article/details/73608613)）
        - suid(set User ID,set UID)的意思是进程执行一个文件时通常保持进程拥有者的UID。然而，如果设置了可执行文件的suid位，进程就获得了该文件拥有者的UID。
        - sgid(set Group ID,set GID)意思也是一样，只是把上面的进程拥有者改成进程组就好了
    - 如果一个文件被设置了suid或sgid位，会分别表现在所有者或同组用户的权限的可执行位上；如果文件设置了suid还设置了x（执行）位，则相应的执行位表示为s(小写)。但是，如果没有设置x位，它将表示为S(大写)。

其中：r-读权限（4），w-写权限（2），x-执行权限（1），-不具有任何权限（0），s-文件被执行时依据who参数指定用户类型设置权限

7-可读可写可执行，6-可读可写，5-可读可执行，4-只读，3-可写可执行，2-可写，1-可执行，其中授权码为奇数的具有执行权限

### 操作

授权基础命令格式：chmod [用户] [操作] [权限] [目标文件]

- 用户：u-文件所有者，g-所有者所在组，o-其他用户，a-所有用户
- 操作：+增加权限，-收回权限，=仅授予权限

示例：

- chmod u+r test.sh 增加所有者对test.sh文件的读权限

- chmod u+rw, g-x, o=x text.sh 增加所有者的读写权限，撤销同组成员的执行权限，将其他用户权限置为只执行


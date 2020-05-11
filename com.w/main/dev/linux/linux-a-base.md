# Linux练习

> dommand [-option] parameter1 parameter2

# 1.显示时间命令

## 1.1主命令：date

1.1.1示例：

```java
date %Y-%m-%d-%H:%M:%s
```

## 1.2主命令：cal(日历)

```java
cal [[month] year]
```

# 2.常用热键

## 2.1tab键

> 命令或文件名补全

## 2.2ctrl+c

> 紧急停止当前执行命令

## 2.3ctrl+d

> 键盘输入结束符

# 3.在线求助命令

## 3.1manPage

### 3.1.1常规man界面操作字符

- /String: 向下查询字符串
- ?String: 向上查询string字符串
- n,N: n向上查询，N向下查询
- q: 结束
- 空格：下翻
- 另外方向键同样有效：上PageUp 下Page Down，首页HOME，最后一页END

### 3.1.2查看相关man文件

- man 1 man

### 3.1.3man简写命令

> 以下命令使用前提是已有whatis数据库，root创建数据库:makewhatis

- whatis相当于man -f
- apropos相当于man -k

## 3.2infoPage

# 4.文本编辑器

## 4.1nano

- 示例：nano text.txt

# 5.正确关机

## 5.1命令列表

- 关机：shutdown或reboot
- 数据写入磁盘：sync
- 管用关机命令：shutdown
- 重启，关机：reboot,halt,poweroff

## 5.2数据写入磁盘：sync

> 建议系统关机前，重启之前多执行几次

# 6.文件

## 6.1权限控制

### 6.1.1chgrp

### 6.1.2chwn

### 6.1.3chmod

# 7.文件夹操作

## 7.1操作

### 7.1.1创建

- mkdir dname
- 创建多层：mkdir -p dname/dname1/dname2

### 7.1.2删除

- rmdir [-p] name

    > 联通上层空目录一起删除（仅删除空目录）

### 7.1.3显示文件或文件夹

- ls

    -a所有文件

    -A所有，不包括.XX/..XX

    -d目录本身

    -f列出不排序结果

    -F提供附加数据结构

    -h文件容量

    -i列出inode号码

    -l列出长数据结构

    -n列出UID，GID

    -r排序结果反向输出

    -R连同子目录一起列出

    -S文件容量大小排序

    -t时间排序

### 7.1.4复制，删除，移动

- cp复制：

    > 源文件两个以上，目标文件需要是目录

- rm删除：

- mv移动：

### 7.1.5取得名称

- basename XXX/XXX

    > 取得文件名

- dirname XX/XXX

    >  取得目录名

7.1.6查看文件内容

- cat,tac,nl

## 7.2权限管理

- 设定权限：mkdir -m 711 test2
- 默认权限：与umask有关


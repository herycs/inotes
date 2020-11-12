# Crontab

定时任务

## 基础操作

### 打开日志

service rsyslog restart

### 重启crontab

service cron restart

### 查看是否运行

pgrep cron

## 所在文件

cat /var/spool/cron/root

## 使用

1）crontab -u /*设定某个用户的cron服务*/

2）crontab -l /*列出某个用户cron服务的详细内容*/

3）crontab -r /*删除某个用户的cron服务*/

4）crontab -e /*编辑某个用户的cron服务*/

参数名称 含义 示例

-l 显示用户的Crontab文件的内容 crontabl –l

-i 删除用户的Crontab文件前给提示 crontabl -ri

-r 从Crontab目录中删除用户的Crontab文件 crontabl -r

-e 编辑用户的Crontab文件 crontabl -e

/etc/crontab文件语法如下：

Minute Hour Day Month Dayofweek command

秒 分钟 小时 天 月 天每星期 命令

每个字段代表的含义及取值范围如下：

Minute ：分钟（0-59），表示每个小时的第几分钟执行该任务

Hour ： 小时（1-23），表示每天的第几个小时执行该任务

Day ： 日期（1-31），表示每月的第几天执行该任务

Month ： 月份（1-12），表示每年的第几个月执行该任务

DayOfWeek ： 星期（0-6，0代表星期天），表示每周的第几天执行该任务

Command ： 指定要执行的命令（如果要执行的命令太多，可以把这些命令写到一个脚本里面，然后在这里直接调用这个脚本就可以了，调用的时候记得写出命令的完整路径）

在这些字段里，除了“Command”是每次都必须指定的字段以外，其它字段皆为可选字段，可视需要决定。对于不指定的字段，要用“*”来填补其位置。同时，cron支持类似正则表达式的书写，支持如下几个特殊符号定义：

“*” ,代表所有的取值范围内的数字；

“/” , 代表”每”（“*/5”，表示每5个单位）；

“-” , 代表从某个数字到某个数字（“1-4”，表示1-4个单位）；

“,” ,分开几个离散的数字；

## 日志启动时间

ununtu 通过调用 run-parts 命令，定时运行四个目录下的所有脚本。

1）/etc/cron.hourly，目录下的脚本会每个小时让执行一次，在每小时的2分钟时运行；

2）/etc/cron.daily，目录下的脚本会每天让执行一次，在每天0点17分时运行；

3）/etc/cron.weekly，目录下的脚本会每周让执行一次，在每周第七天的3点56分时运行；

4）/etc/cron.mouthly，目录下的脚本会每月让执行一次，在每月19号的5点32分时运行

> 转载：[简书Ten_Minutes](https://www.jianshu.com/p/d6d8d9f7f60c)


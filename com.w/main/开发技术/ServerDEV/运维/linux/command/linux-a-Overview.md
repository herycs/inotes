# Overview

## 文件管理

### ls

ls 命令将每个由 Directory 参数指定的目录或者每个由 File 参数指定的名称写到标准输出，以及您所要求的和标志一起的其它信息。

### lsof

lsof（list open files）是一个列出当前系统打开文件的工具。

### find

文件查找

### locate

Linux locate命令用于查找符合条件的文档，他会去保存文档和目录名称的数据库内，查找合乎范本样式条件的文档或目录。

locate 与 find 不同: find 是去硬盘找，locate 只在 /var/lib/slocate 资料库中找。

locate 的速度比 find 快，它并不是真的查找，而是查数据库，一般文件数据库在 /var/lib/slocate/slocate.db 中，所以 locate 的查找并不是实时的，而是以数据库的更新为准，一般是系统自己维护，也可以手工升级数据库(updatedb)

### mdir

Linux mdir命令用于显示MS-DOS目录。

### grep

Linux grep 命令用于查找文件里符合条件的字符串。

## 移动文件

### mv

Linux mv 命令用来为文件或目录改名、或将文件或目录移入其它位置。

### cp

Linux cp命令主要用于复制文件或目录。

## 文件内容

### cat

cat 命令用于连接文件并打印到标准输出设备上。

### less

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。

### more

Linux more 命令类似 cat ，不过会以一页一页的形式显示，更方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能（与 vi 相似），使用中的说明文件，请按 h 。

### diff

Linux diff命令用于比较文件的差异。

diff以逐行的方式，比较文本文件的异同处。如果指定要比较目录，则diff会比较目录中相同文件名的文件，但不会比较其中子目录。

### awk

AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。

### sort

Linux sort命令用于将文本文件内容加以排序。

sort可针对文本文件的内容，以行为单位来排序。

## 文件传输

### ftp

Linux ftp命令设置文件系统相关功能。

## 文件信息修改

### touch	

Linux touch命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

## 权限

### chmod

Linux/Unix 的文件调用权限分为三级 : 文件拥有者、群组、其他。利用 chmod 可以藉以控制文件如何被他人所调用。

## 显示进程

### ps

Linux ps命令 Linux 命令大全 Linux ps命令用于显示当前进程 (process) 的状态

## 磁盘使用

### du

全称是 disk usage，du会显示指定的目录或文件所占用的磁盘空间。

### df

df命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

## 网络配置

### ping

ping命令用于检测主机

### ifconfig

ifconfig是[linux](https://baike.baidu.com/item/linux/27050)中用于显示或配置网络设备（[网络接口卡](https://baike.baidu.com/item/网络接口卡/9764230)）的命令，英文全称是network interfaces configuring。

### netstat

netstat命令的功能是显示网络连接、路由表和网络接口信息

### firewall

防火墙控制命令

### route

route命令用来显示并设置Linux内核中的网络路由表，route命令设置的路由主要是静态路由。

### top

top命令可以实时动态地查看系统的整体运行情况，是一个综合了多方信息监测系统性能和运行信息的实用工具。

## 定时任务

### crontab

crontab命令被用来提交和管理用户的需要周期性执行的任务
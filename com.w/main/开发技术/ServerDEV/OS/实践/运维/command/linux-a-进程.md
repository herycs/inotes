# 进程

## 进程启动路径

在linux下查看进程大家都会想到用 ps -ef|grep XXX

可是看到的不是全路径，怎么看全路径呢？

每个进程启动之后在 /proc下面有一个于pid对应的路径

例如：ps -ef | grep python

显示：oracle  4431 4366 0 18:56 pts/2  00:00:00 python Server.py

4431就是进程号

到/proc/4431下，ls -l 会看到：


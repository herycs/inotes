# Ubuntu一

# 初始配置

> 查看linux版本：cat /proc/version

## 网络环境

### 配置

以上版本

- 网络配置文件在：

  ```less
  /etc/netplan/50-cloud-init.yaml
  /etc/network/interfaces
  ```
```

- 50-could-init.yaml配置范例

    ```less
    network:
      ethernets:
            ens33:
                addresses:
                - 192.168.43.200/24
                gateway4: 192.168.43.0
                nameservers: 
                    address: [8.8.8.8 8.8.4.4]
                    search: []
                optional:true
        renderer: networkd
        version: 2
```

- DNS：

  ```less
  /etc/systemd/resolved.conf
  ```

- 添加默认网关：

  ```less
    sudo route add default gw 192.168.1.1
  ```

### 相关命令

- 重启网络服务：

  ```less
  sudo /etc/init.d/networking force-reload
  sudo /etc/init.d/networking restart
  ```
  
- 重新启停以太网卡：

  ```less
  sudo ifconfig eth0 down
  sudo ifconfig eth0 up
  ```

## 配置网卡

获取UUID

```shell
 uuidgen eth1
```



### 资源

- 镜像地址：
  - https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

### 操作

- 修改并编辑：

  ```less
  /etc/apt/sources.list
  ```

  

- 更新源： 

  ```less
  sudo apt-get update
  ```

# 查看信息

## 查看版本信息

- 统的版本信息

  ```less
  cat /proc/version
  ```

- 显示linux的内核版本和系统是多少位的

  ```less
  uname -a
  ```

- 简介显示

  ```less
  lsb_release -a
  ```

## 查看文件

### 包文件

- 命令

  ```less
  apt-get install apt-file
  
  apt-file search 文件名
  
  apt-get install 
  ```

### 查看库文件

- 命令

  ```less
  #查找文件：
  sudo find /usr/ -name fileName
  ```

- 安装

  ```less
  #添加搜索路径
  sudo apt-get install apt-file
  apt-file update
  apt-file search fileName
  sudo apt-get install fileName:version
  ```

### 查看文件内容

#### 指定查看文件行数

> tail 是指定行数到结尾
> head是开头到指定行数
> 	+数字 代表整数第几行， -数字代表倒数第几行。

- tail

  ```less
  tail date.log           输出文件末尾的内容，默认10行
  tail -20  date.log      输出最后20行的内容
  tail -n -20  date.log   输出倒数第20行到文件末尾的内容
  tail -n +20  date.log   输出第20行到文件末尾的内容
  tail -f date.log        实时监控文件内容增加，默认10行。
  ```

- head

  ```less
  head date.log         输出文件开头的内容，默认10行
  head -15  date.log   输出开头15行的内容
  head -n +15 date.log 输出开头到第15行的内容
  head -n -15 date.log 输出开头到倒数第15行的内容
  ```

- sed

  ```less
  sed -n "开始行，结束行p" 文件名
  #输出第70行到第75行的内容
  sed -n '70,75p' date.log
  #输出第6行 和 260到400行
  sed -n '6p;260,400p; ' 文件名
  #输出第5行
  sed -n 5p 文件名                    
  ```

# 安装

## 安装GCC

使用命令直接安装
- 建议先配置镜像
- 使用安装命令

手工编译安装GCC

- 各个软件包的安装顺序是m4 --> gmp --> mpfr --> mpc --> gcc
- 下载源文件--->解压--->编写makefile文件--->编译--->检查

### 命令

- 安装：

  ```less
  sudo apt-get install gcc
  ```

- 生成file文件：

  ```less
  sudo ~/gcc_depend/m4-1.4.16/configure --prefix=/usr/local/m4-1.4.16 # 通过configure脚本来生成Makefile
  ```

- 编译：

  ```less
  - sudo make 
  - sudo make install
  ```

- 检查

  ```less
  sudo make check
  ```

相关命令：

```less
sudo apt-get update  更新源
sudo apt-get install package 安装包
sudo apt-get remove package 删除包
sudo apt-cache search package 搜索软件包
sudo apt-cache show package  获取包的相关信息，如说明、大小、版本等
sudo apt-get install package --reinstall  重新安装包
sudo apt-get -f install  修复安装
sudo apt-get remove package --purge 删除包，包括配置文件等
sudo apt-get build-dep package 安装相关的编译环境
sudo apt-get upgrade 更新已安装的包
sudo apt-get dist-upgrade 升级系统
sudo apt-cache depends package 了解使用该包依赖那些包
sudo apt-cache rdepends package 查看该包被哪些包依赖
sudo apt-get source package  下载该包的源代码
sudo apt-get clean && sudo apt-get autoclean 清理无用的包
sudo apt-get check 检查是否有损坏的依赖
unzip filename  解压zip文件
```

# 卸载

## 查询

### rpm

- 查询所有软件包

  ```less
  rpm -q -a
  ```

- 解压的软件包

  ```less
  make uninstall
  ```

# 进程管理

## 查找进程

- 查找

  ```less
  #查找对应的进程信息
  ps -ef | grep firefox
  
  #查找对应进程PID
  pgrep firefox
  pidof firefox-bin
  ```

- 杀死进程

  ```less
  kill -s 9 1827
  ```

# 文件管理

## 文件重命名

- mv或cp命令

# 用户管理

## root

- ubuntu系统root用户

  ```less
  sudo passwd root
  ```

# 网关管理

##  命令

> linux使用默认路由会选择metric值小的

> route命令添加的网关Metric=0，优先级是0
>
> nmtui设置的网关优先级是100

- 查看网关

  ```less
  #路由信息
  route -n
  #本地ip信息
  ip route show
  #网关及网络设置
  netstat -r
  ```

- 删除网关

  ```less
  route del default gw 10.1.1.254
  ```

- 添加网关

  ```less
  route add default gw 192.168.43.1
  #添加并设置临时权重，重启后消失
  route add default gw 10.1.1.254 metric 101
  #永久设置权重
  nmtic 命令
  配置文件
  ```

  


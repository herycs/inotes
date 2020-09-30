# VM-net

## 基础

> 首先需要了解下VM的几种网络连接方式

### 桥接

```shell
主机网卡 --- VMnet0(虚拟机交换机)
  |              |
  |              |
主机         {虚拟机群}
```

虚拟机通过虚拟交换机从以太网上获取网络信息，并向指定虚拟机提供服务

主机网卡和虚拟交换机通过虚拟网桥进行数据传输

**要求**

IP同主机网段

子网掩码，网关，DNS同主机

**配置**

1. win查看本地连接网络属性以获取（主机IP，默认网关，DNS）

2. vim /etc/sysconfig/network-script/ifcfg-eth33

    ```shell
    修改或添加
    BOOTPROTO=static
    添加
    IPADDR=192.168.43.100
    METMASX=255.255.255.0
    GATEWAY=192.168.43.2
    ```

3. 重新启动

    /etc/init.d/network restart

### NAT

> 地址转换模式

依据虚拟NAT设备和DHCP服务器

模式

```
主机网卡---虚拟NAT---     ---虚拟DHCP
  |                 |    |
  |                 VM net8
  |                 |    |
主机---VM Adapter8---     ---{虚拟机群}
```

以上，虚拟机和外网的通信采用的是NAT直接和网卡交互，期间绕过了主机，故不启动VM Adapter8同样可以联网，但要实现和主机的交互就得依靠VM Adapter8来提供服务

**配置**

1. 配置本地适配器网络地址
2. 配置VM网络模式为NAT模式，设置NAT设置和DHCP设置
3. 配置虚拟机网络模式为NAT模式

### Host-Only

可理解为出去了NAT中的虚拟NAT

模式

```shell
主机网卡                          ---虚拟DHCP
  |                              |
  |     VM Adapter1---VM net1(虚拟交换机)
  |      |                       |
主机------                    {虚拟机群}
```

其中：

- 初始策略为，虚拟机群**只能与主机**通信，通过主机实现和外网的联通
- 若希望虚拟机群拥有和外网联通的能力，则可将主机网卡共享给VMnet1

## 前情提要

无论是采用以上哪种方式，我们的虚拟主机都是需要通过VM提供的网络服务进行通讯，而如何和外网连通和主机网卡连通这是VM的事，配置文件时虚拟机内部的配置文件目标是链接到VM当前网络环境下的网络设备上，故虚拟机的配置目标是能和VM提供的网络服务打通。

简单来说：

- VM的网络配置器：**VM软件**和**主机的网卡**打通以支持**对外网**的访问，此过程成功后连上VM就是连通了外网
- 虚拟机和VM网络连通：将**虚拟机的网卡**和**VM的网络配置器**连通得到*VM提供*的**外网访问**能力，
- 虚拟机和主机打通：以提供**虚拟机**和**主机**间的交互，此过程成功后就可和主机进行交互

## 配置

### 配置网络适配器

> 操作环境：Windows 网络适配器

1. 打开本地网络适配器（此处可以注意下当前电脑连接的网络名称）

2. 配置对应的适配器：属性--->IPv4.属性--->配置虚拟机打算用的网络地址

    - VMnet8（采用NAT配置网络）

        例如：配置为IP=192.168.43.1，NETMASK=255.255.255.0，GATEWAY=192.168.43.2

    - VMnet1（桥接模式，此处可不用设置）

### VM网络网络编辑器

> 操作环境：VM软件网络编辑器
>
> 为了避免后期麻烦，建议一次将VM0，VM1，VM8的设置设置好

1. 配置**桥接模式**设置VM0：

    (若没有就自己添加个)

    桥接到本地和外网的链接网络（例如当前是Wifi2连接的，则选用Wifi2，具体名称可在win本地网络适配器上看到）

2. 配置**仅主机模式**设置VM1：

    仅主机，可以开启DHCP

3. 配置**NAT模式**设置VM8：

    打开VM的网络编辑器，选择VM8

    - NET配置：

        配置**网关**地址（同1中配置网络编辑器的网段，此处应为：192.168.43.0）

    - DHCP配置：

        划分IP地址**范围**，例如：192.168.43.100-192.168.43.165

### 虚拟机网络配置

> 操作环境：linux虚拟机
>
> 进入虚拟机修改网络配置文件，进行网络信息的配置

#### centos

**ip文件修改**

vim /etc/sysconfig/network-scripts/ifcfg-eth33

```shell
文件中添加
IPADDR=192.168.43.100
NETMASK=255.255.255.0
GATEWAY=192.168.43.2
修改
#地址分配策略
BOOTRPOTO=static
#设置网卡开机自启动
ONBOOT=yes
```

重启net服务

systemctl restart network

**dns文件修改**

vim /etc/resolv.conf

```shell
添加国内dns
nameserver 114.114.114.114
nameserver 114.114.115.115
# 阿里
nameserver 223.5.5.5
nameserver 223.6.6.6
# 百度
nameserver 186.76.76.76
```

经历以上流程基本网络也就通了，若是还有问题可检查下上述过程的操作，或者直接按照下述配置配置网络

> 基本信息如下：
>
> win适配器
>
> ​	VM8 adapter
>
> ​	ip 192.168.43.1
>
> ​	mask 255.255.255.0
>
> ​	gateway 192.168.43.2
>
> VM 网络编辑器
>
> ​	修改Vmnet8
>
> ​	nat的网关地址为：192.168.43.0
>
> ​	dhcp：设置网络范围为（这个无所谓）：192.168.43.99-192.168.43.176
>
> linux虚拟机网络文件修改
>
> ​	vim /etc/sysconfig/network-scripts/ifcfg-eth33
>
> ​	vim /etc/resolv.conf

#### ubuntu

> 流程同上区别就是操作文件位置不同，具体检索以上版本对应的配置文件位置

## 问题

### 本地连接虚拟机失败

开启ssh

sudo /etc/init.d/ssh start

端口放通

ufw allow 22/tcp

### 虚拟机ping失败

1. ping 本机失败

    查看网络配置

    ```shell
    ip配置
    vim /etc/network/interfaces
    vim /etc/netplan/50-...
    增加：
    auto eth33
    iface eth33 inet static
    address 192.168.43.100
    netmask 255.255.255.0
    gateway 192.168.43.2
    
    DNS配置：
    vim /etc/rsolve.config
    增加
    nameserver 8.8.8.8
    执行
    resolvconf -u
    ```

2. ping 网关失败

    ```shell
    查看当前网络设置
    route -n
    添加网关
    sudo route add default gw 192.168.43.1
    ```

3. ping 主机失败，但主机ping虚拟机成功（一般用不到这么麻烦，检查以上几步，实在麻烦换个centos安装时按照教程配置好即可，当然前提是网络适配器和VM网络编辑器的配置没有问题）

    win防火墙

4. ping 外网失败

    校验以上几步  
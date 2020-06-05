# docker

## 基础操作

docker run xxx

docker stop

docker ps -a

删除镜像

docker rmi

## 容器操作

**搜索镜像**

docker search

### 下载镜像

dcker pull

```shell
docker pull elasticsearch:5.6.8
docker pull mobz/elasticsearch-head:5
docker pull gogs/gogs
docker pull grafana/grafana
docker pull rabbitmq
docker pull mysql
docker pull redis
docker pull tutum/influxdb
docker pull rancher/server
docker pull mongo
docker pull registry
docker pull google/cadvisor
docker pull jenkins
```

### 制作容器

> 利用已有的

```shell
//mysql
docker run -di --name=[innerMysql] -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123 [imageName] : [version](已经是最新版则不需要添加此参数)
不修改配置文件，mysql默认监听3306
docker run -d --name selfdefinename -p 10086:3306 -e MYSQL_ROOT_PASSWORD=rw mysql:5.6
//redis 
docker run -di --name=itaobao_resdis -p 6379:6379 redis
//mongo
docker run -di --name=itaoabo_mongo -p 27017:27017 mongo
//es
docker run -dit --name=itaobao_es -p 9200:9200 -p 9300:9300 elasticsearch:5.6.8
//es-head
docker run -di --name=itaoabo_head -p 9100:9100 mobz/elasticsearch-head:5
//rabbitmq
docker run -di --name=itaobao_mq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq
//registry
docker run -di --name=itaobao_registry -p 5000:5000 registry
//gogs
docker run -di --name=itaobao_gogs -p 10022:22 -p 3000:3000 -v /var/gogsdata:/data gogs/gogs
//jenkins，可在线下载也可上传
wget https://pkg.jenkins.io/redhat/jenkins-2.156-1.1.noarch.rpm
rpm -ivh jenkins-2.156-1.1.noarch.rpm
//rancher
docker run -d --name=itaobao_rancher --restart=always -p 9090:8080 rancher/server
//influxdb
docker run -d --name=itaobao_influxdb -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 tutum/influxdb
//cadvisor
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --link itaobao_influxdb:itaobao_influxdb --name=cadvisor google/cadvisor -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=itaobao_influxdb:8086
//grafana
docker run -d -p 3001:3000 -e INFLUXDB_HOST=itaobao_influxdb -e INFLUXDB_PORT=8086 -e INFLUXDB_NAME=cadvisor -e INFLUXDB_USER=cadvisor -e INFLUXDB_PASS=cadvisor --link itaobao_influxdb:itaobao_influxdb --name grafana grafana/grafana
//发布微服务
docker run -di --name=itaobao_config -p 12000:12000 122.51.8.208:5000/itaobao_config:1.0-SNAPSHOT
docker run -di --name=itaobao_eureka -p 6868:6868 122.51.8.208:5000/itaobao_eureka:1.0-SNAPSHOT
```

### es

```shell
安装eS需要做以下处理
1.解除9300端口限制

先安装一次,随后将其文件下拉到本地
docker run -di --name=itaobao_es -p 9200:9200 -p 9300:9300 elasticsearch:5.6.8
docker cp itaobao_es:/usr/share/elasticsearch/config/elasticsearch.yml /usr/local/config/elasticsearch.yml
docker stop itaobao_es
docker rm itaoabo_es

建立文件映射
docker run -dit --name=itaobao_es -p 9200:9200 -p 9300:9300 -v /usr/local/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:5.6.8

取消修改文件.yml中的注释
transport.host:0.0.0.0（限制访问IP）

2.调整参数，保证es正常运行

/etc/security/limits.conf追加两行配置，修改其单个进程能申请的软硬件限制
soft nofile 65536
hard nofile 65536

/etc/sysctl.conf追加
vm.max_map_count=655360

执行使得参数生效
sysctl -p
重启虚拟机，再次启动容器

3.添加ik分词器
cp ik itaobao_es:/usr/share/elasticsearch/plugins
重启容器

4.跨域
追加到共享文件：/user/local/config/elasticsearch.yml
http.cors.enable: true
http.cors allow-origin: "*"
```

### mq

```shell
rabbitmq
安装完没有图形界面,其本身包含，但出于关闭状态，需要打开
进入容器中
docker exec -it itaobao_mq bash
执行（打开图形界面）
rabbitmq-plugins enable rabbitmq_management
```

### registry

```shell
registry
访问地址
IP:5000/v2/_catalog
vim /etc/docker/daemon.json
追加
{"insecure-registries":["服务器IP:5000"]}
推送镜像至仓库
docker tag [imageName] [ServerIP]:5000/[newName]
```

### jenkins

```shell
vi /etc/sysconfig/jenkins
//修改端口账号,jenkins需要执行一些操作，直接授权root用户会比较方便，或者也可以选择去授权给jenkins用户
JENKINS_USER="root"
JENKINS_PORT="8088"
service jenkins start
/var/lib/jenkins/secrets/initialAdminPassword
```

> 自己创建镜像

```shell
dockerfile构建镜像
docker build -t='imageName' [sourcePath]
```

## 登录容器

docker exec -it itaobao_mysql bash

## 打包镜像

docker commit -a "king西阳" -m "a new image" [容器名称或id] [打包的镜像名称]:[标签]

## 端口映射

查找容器ID

docker inspect itaobao_mysql | grep Id

**进到/var/lib/docker/containers 目录下找到与 Id 相同的目录，修改 hostconfig.json 和 config.v2.json文件**：

## 批量操作

> 其实就是使用命令实现对id的批量读取，并添加到操作指令中

### ps

将第一列的参数打印出来

docker ps -a | awk '{print $1}'

1. 批量执行命令

    docker ps -a | awk '{print $1}'|xargs docker stop

    Error response from daemon: No such container: CONTAINER

    过滤Exited状态容器

    docker rm `docker ps -a|grep Exited|awk '{print $1}'`

2. sudo docker rm $(sudo docker ps -qf status=exited)

3. Docker 1.13版本以后，可以使用 docker containers prune 命令，删除孤立的容器。

    sudo docker container prune

### 镜像批量操作

> 经过对上面的操作，类比操作方式，发现莫名出现的镜像都是TAG为none的

docker images | grep none | awk '{print $3}'

docker rm -i `docker images | grep none | awk '{print $3}'

## docker

### 安装

**查看系统版本**

docker官方建议linux3.8以上

uname -a

**更新工具**

yum update

**安装软件包**

yum install -y yum-utils device-mapper-persistent-data lvm2

**设置源**

> 国内建议阿里云的仓库

yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

**选择版本**

yum list docker-ce --showduplicates | sort -r

**安装**

yum install docker-ce-18.03.1.ce

### 启动

systemctl start docker

加入开机自启动：systemctl enable  docker

## MAVEN构建镜像

**添加许可**

```shell
vim /lib/systemd/system/docker.service
ExecStart
追加(IP远程访问Maven的许可)
-H tcp://1.84.16.0:2375 -H unix:///var/run/docker.sock
重载配置
systemctl daemon-reload
```

**构建并上传**

mvn clean package docker:build -DpushImage

## 报错

### net

1.WARNING: IPv4 forwarding is disabled. Networking will not work.  

```shell
vim  /usr/lib/sysctl.d/00-system.conf

添加
net.ipv4.ip_forward=1

重启服务
systemctl restart network
```

### jenkins

2.jenkins报错: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)

> 凡事尤其原由，若出现了错误日志是最好的提示方式

```shell
1.先检查自己的java命令正常不，不正常先处理java，例如下列数据中/usr/bin/java这个命令能不能正常运行，这是jenkins用来运作的一个基础命令，若此命令不可运行，自然会出错
vim /etc/rc.d/init.d/jenkins
//检查文件
candidates="
/etc/alternatives/java
/usr/local/jdk/jdk1.7.0_71/bin/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/bin/java
2.若java命令正常，/usr/bin/java命令也正常再继续看报错或者当前状态
systemctl status jenkins
一句一句读提供的信息
例如我之前便忽略这个信息
Starting Jenkins Jenkins requires Java8 or later, but you are running 1.7.0_71-b14 from /usr/local/jdk/jdk1.7.0_71/jre
jenkins要求jdk1.8,我的配置是jdk1.7=>解决办法是重装jdk,重新运行后成功
```

### mysql远程连接10060

1.服务是否正常启动

2.端口是否开放，服务器端口，云端口

云端口若为：::::/0可以改成0.0.0.0/0试一下

3.检查授权

use mysql;

select user, host from mysql

更新授权

## 卸载

```shell
-- 查询已安装的
rpm -qa| grep mysql
-- 卸载已安装的
rpm -e --nodeps [此处跟你查询出来的mysql-XXX]
```

## jdk

```
ln -s /usr/local/jdk/jdk1.8.0_171/bin/java /usr/bin/java
ln -s /usr/local/jdk/jdk1.8.0_171/bin/javac /usr/bin/javac
ln -s /usr/local/jdk/jdk1.8.0_171/bin/jar /usr/bin/jar
```


# docker

## 安装

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

### 修改配置

```shell
vim /lib/systemd/system/docker.service
systemctl daemon-reload 
systemctl restart docker
```

### 启动

systemctl start docker

加入开机自启动：systemctl enable  docker

## 基础操作

### 打包镜像

docker commit -a "king" -m "a new image" [容器名称或id] [打包的镜像名称]:[标签]

### 批量操作

> 其实就是使用命令实现对id的批量读取，并添加到操作指令中

将第一列的参数打印出来

docker ps -a | awk '{print $1}'

1. 批量执行命令

    ```shell
    docker ps -a | awk '{print $1}'|xargs docker stop
    # 过滤Exited状态容器
    docker rm `docker ps -a|grep Exited|awk '{print $1}'`
    ```

2. sudo docker rm $(sudo docker ps -qf status=exited)

3. Docker 1.13版本以后，可以使用 docker containers prune 命令，删除孤立的容器。

    ```shell
    sudo docker container prune
    ```

### 镜像批量操作

> 经过对上面的操作，类比操作方式，发现莫名出现的镜像都是TAG为none的

```shell
docker images | grep none | awk '{print $3}'
docker rmi `docker images | grep none | awk '{print $3}'`
```

## 镜像管理

```shell
# 搜索
docker search {查找关键字，例如：mysql}
# 下载
docker pull {镜像名称}:{版本号}
# 删除
docker rmi {镜像ID}
```

- 一些下载示例

    ```shell
    docker pull elasticsearch:5.6.8
    docker pull mobz/elasticsearch-head:5
    docker pull gogs/gogs
    docker pull grafana/grafana
    docker pull rabbitmq
    docker pull mysql:5.7
    docker pull redis
    docker pull tutum/influxdb
    docker pull rancher/server
    docker pull mongo
    docker pull registry
    docker pull google/cadvisor
    docker pull jenkins
    ```

## 容器管理

```shell
# 打印所有
docker ps -a
# 打印在运行
docker ps
# 停止
docker stop {容器ID}
# 启动
docker start {容器ID}
```

### 端口映射

查找容器ID

docker inspect itaobao_mysql | grep Id

**进到/var/lib/docker/containers 目录下找到与 Id 相同的目录，修改 hostconfig.json 和 config.v2.json文件**

### 制作容器

> 利用已有的制作容器

```shell
docker run -di --name={容器名称} -p {系统端口}:{容器内端口} {容器名称}:{版本}
```

### 进入容器

```shell
docker exec -it {容器名称} /bin/bash
```

### 操作示例

#### mysql

```shell
# 不修改配置文件，mysql默认监听3306
docker run -d --name itaobao_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456.sql mysql

docker run -p 3306:3306 --name itaobao_mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456.sql \
-idt mysql:5.7 \
--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

#### redis

```shell
docker run -di --name=itaobao_resdis -p 6379:6379 redis
```

#### redis

```shell
//mongo
mkdir -p /usr/mongo/data/db
docker run -di --name=itaoabo_mongo -p 27017:27017 mongo
docker run -di --name=itaoabo_mongo -v /usr/mongo/data/db:/data/db -p 27017:27017 mongo
```

#### es

```shell
//es
docker run -dit --name=itaobao_es -p 9200:9200 -p 9300:9300 elasticsearch:5.6.8
//es-head
docker run -di --name=itaoabo_head -p 9100:9100 mobz/elasticsearch-head:5
```

#### rabbitMQ

```shell
docker run -di --name=itaobao_mq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq
```

#### registry

```shell
//registry
docker run -di --name=itaobao_registry -p 5000:5000 registry
docker run -di --name=itaobao_registry -v /var/lib/registry:~/registry/data -p 5000:5000 registry
```

#### gogs

```shell
//gogs
docker run -di --name=itaobao_gogs -p 10022:22 -p 3000:3000 -v /var/gogsdata:/data  gogs/gogs
```

#### jenkins

> 官网建议至少有IG空间给jenkins运行

```shell
//jenkins，可在线下载也可上传
wget https://pkg.jenkins.io/redhat/jenkins-2.156-1.1.noarch.rpm
rpm -ivh jenkins-2.156-1.1.noarch.rpm
```

#### rancher 

```shell
//rancher
docker run -d --name=itaobao_rancher --restart=always -p 9090:8080 rancher/server
```

#### influxdb

```shell
//influxdb
docker run -d --name=itaobao_influxdb -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 tutum/influxdb
```

#### cadvisor

```shell
//cadvisor
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --link itaobao_influxdb:itaobao_influxdb --name=cadvisor google/cadvisor -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=itaobao_influxdb:8086
```

#### grafana

```shell
//grafana
docker run -d -p 3001:3000 -e INFLUXDB_HOST=itaobao_influxdb -e INFLUXDB_PORT=8086 -e INFLUXDB_NAME=cadvisor -e INFLUXDB_USER=cadvisor -e INFLUXDB_PASS=cadvisor --link itaobao_influxdb:itaobao_influxdb --name grafana grafana/grafana
```

### 安装附加操作

#### es

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

#### mq

```shell
rabbitmq
安装完没有图形界面,其本身包含，但出于关闭状态，需要打开
进入容器中
docker exec -it itaobao_mq bash
执行（打开图形界面）
rabbitmq-plugins enable rabbitmq_management
```

#### registry

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

#### jenkins

```shell
vi /etc/sysconfig/jenkins
//修改端口账号,jenkins需要执行一些操作，直接授权root用户会比较方便，或者也可以选择去授权给jenkins用户
JENKINS_USER="root"
JENKINS_PORT="8088"
service jenkins start
/var/lib/jenkins/secrets/initialAdminPassword
```

**报错**

- jenkins报错: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)

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

> 自己创建镜像

```shell
dockerfile构建镜像
docker build -t='imageName' [sourcePath]
```

## MAVEN构建镜像

### **添加许可**

> IP设置为：0.0.0.0=>允许任意IP发起推送请求
>
> IP设置为指定IP：x.x.x.x=>允许此IP发起推送请求（例如设置为：192.168.42.101）

```shell
# 打开文件
vim /lib/systemd/system/docker.service
# 追加(IP远程访问Docker的许可)
ExecStart
-H tcp://{此处替换为要向服务器发起推送的IP}:2375 -H unix:///var/run/docker.sock
# 重载配置
systemctl daemon-reload
systemctl restart docker
```

### **构建并上传**

```shell
# 命令行执行
mvn clean package docker:build -DpushImage
```

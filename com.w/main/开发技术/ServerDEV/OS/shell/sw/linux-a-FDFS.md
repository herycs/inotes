# FastDFS

# 1.分布式文件系统

## 1.1前置资源文件

- 提前下载
  - fastdfs-nginx-module_v1.16.tar.gz
  - FastDFS_Java_Doc_v1.20.tar.gz
  - FastDFS_v5.05.tar.gz
  - libfastcommonV1.0.7.tar.gz

- fastdfs 需要一个网络驱动包  libevent，系统中下载

  - libevent-1.2.tar.gz

- 提供以下直接下载命令

  ```less
  #libfastcommon
  wget https://github.com/happyfish100/libfastcommon/archive/master.zip
  wget https://github.com/happyfish100/libfastcommon/archive/V1.0.38.tar.gz
  #fastDFS
  wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz
  ```

  

# 2.安装

## 2.1libevent安装

> fastDFS 版本5之后的可以跳过此步

- 下载：

  ```less
  wget http://www.monkey.org/~provos/libevent-1.2.tar.gz
  ```

- 安装：

  ```less
  # tar zxvf libevent-1.2.tar.gz
  # cd libevent-1.2
  # ./configure –prefix=/usr
  # make
  # make install
  ```

## 2.2其他安装

## 2.2.1libfastcommon安装

- 下载命令

  ```less
  wget https://github.com/happyfish100/libfastcommon/archive/V1.0.38.tar.gz
  ```

- 安装

  ```less
  cd /usr/local
  tar -zxvf libcommonXXX
  cd libcommonXXX
  ./make.sh
  ./makesh install
  ```

- 文件拷贝

  ```less
  libfastcommon 安装好后会自动将库文件拷贝至/usr/lib64 下，由于 FastDFS 程
  序引用 usr/lib 目录所以需要将/usr/lib64 下的库文件拷贝至/usr/lib 下
  ```

  

## 2.2.2tracker安装

- 下载

  ```less
  wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz
  ```

-  安装

  ```less
  tar -zxvf FastDFS_v5.05.tar.gz
  cd FastDFS
  ./make.sh 编译
  ./make.sh install 安装
  ```

- 拷贝

  ```less
  cd conf/
  cp client.conf http.conf mime.types storage.conf storage_ids.conf  tracker.conf /etc/fdfs
  ```

- 配置

  ```less
  cd /etc/fdfs
  
  #拷贝文件
  cp tracker.conf.sample tracker.conf
  
  修改 tracker.conf
  vi tracker.conf
  base_path=/home/yuqing/FastDFS	改为：base_path=/home/FastDFS
  #配置 http 端口：
  http.server_port=80
  ```

- 启动

  ```less
  /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
  ```

  - 启动报错

    ```less
    #问题：
    	/home/yuqing/fastdfs" can't be accessed, error info: No such file or directory ...
    #解决方法：
    	mkdir -p /home/yuqing/fastdfs
    ```

  - 检验

    ```less
    #命令：
    	ps -ef |grep track |grep -v 'grep'
    ```

## 2.2.3storage安装

> 只有一个服务器时，省略下述前三步执行，仅修改配置即可

- 执行2.1

- 执行2.2.1

- 执行2.2.2

  - 下载
  - 安装
  - 拷贝

- 配置

  ```less
  #修改 storage.conf
  vi storage.conf
  
  group_name=group1
  
  base_path=/home/yuqing/FastDFS 改为：base_path=/home/ fastdfs
  
  store_path0=/home/yuqing/FastDFS 改为：store_path0=/home/fastdfs/fdfs_storage
  #如果有多个挂载磁盘则定义多个 store_path，如下
  #store_path1=.....
  #store_path2=......
  
  #配置 tracker 服务器:IP
  tracker_server=192.168.101.3:22122
  #如果有多个则配置多个 tracker
  tracker_server=192.168.101.4:22122
  
  #配置 http 端口
  http.server_port=80
  ```

- 启动

  ```less
  /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
  ```

## 2.3通信测试

- 命令

  ```less
  fdfs_monitor /etc/fdfs/storage.conf
  ```

# 3.FastDFS 和 nginx 整合

## 3.1nginx代理

- 新建nginx-fdfs.conf

  ```less
  #storage 群 group1 组
  upstream storage_server_group1{
   	server 192.168.43.100:81 weight=10;
  }
  
  #storage 群 group2 组
  #upstream storage_server_group2{
  #	server 192.168.101.7:80 weight=10;
  #	server 192.168.101.8:80 weight=10;
  #}
  
  server {
      
      listen 80;
      server_name ccc.test.com;
      
      location /group1{
          proxy_redirect off;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://storage_server_group1;
      }
      
  #    location /group2{
  #        proxy_redirect off;
  #        proxy_set_header Host $host;
  #        proxy_set_header X-Real-IP $remote_addr;
  #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  #        proxy_pass http://storage_server_group2;
  #    }
  }
  ```

  

## 3.2nginx安装(storage服务器)

- 下载

  ```less
  #nginx
  wget http://nginx.org/download/nginx-1.11.8.tar.gz
  #插件
  wget http://jaist.dl.sourceforge.NET/project/fastdfs/FastDFS%20Nginx%20Module%20Source%20Code/fastdfs-nginx-module_v1.16.tar.gz
  ```

### 3.2.1nginx-moudle

- 解压nginx插件

- 进入nginx插件目录

  ```less
  cd FastDFS-nginx-module/src
  ```

- 修改nginx插件配置文件

  ```less
  #修改 config 文件将
  #修改：去掉local、因为实际安装fastdfs时、是放到了/usr/include下
  1. CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
  -> CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
  
  2.  CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"
  -> CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"
  ```

- 拷贝并修改插件文件

  ```less
  #将 FastDFS-nginx-module/src 下的 mod_FastDFS.conf 拷贝至/etc/fdfs/下方便查找
  cp /home/nj/build/fastdfs-nginx-module/src/mod_fdfs.conf /etc/fdfs
  
  #修改文件od_FastDFS.conf
  vi /etc/fdfs/mod_fdfs.conf
  
  base_path=/usr/local/fastdfs
  tracker_server=192.168.1.114:22122
  url_have_group_name = true
  store_path0=/usr/local/fastdfs/storage
  ```

- 拷贝文件

  ```less
  #将libfdfsclient.so 拷贝至/usr/lib 下
  cp /usr/lib64/libfdfsclient.so /usr/lib/
  ```

- 创建目录

  ```less
  mkdir -p /var/temp/nginx/client
  ```

## 3.2.2nginx安装

- 配置nginx

  ```less
  #回到nginx的解压目录
  cd ../nginx-1.11.8
  ./configure \
  --prefix=/usr/local/nginx \
  --pid-path=/var/run/nginx/nginx.pid \
  --lock-path=/var/lock/nginx.lock \
  --error-log-path=/var/log/nginx/error.log \
  --http-log-path=/var/log/nginx/access.log \
  --with-http_gzip_static_module \
  --http-client-body-temp-path=/var/temp/nginx/client \
  --http-proxy-temp-path=/var/temp/nginx/proxy \
  --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
  --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
  --http-scgi-temp-path=/var/temp/nginx/scgi \
  --add-module=/usr/local/fastdfs-nginx-module/src
  
  #执行安装
  make
  make install
  ```

### 3.2.3nginx配置文件

- 新建nginx配置文件nginx-fdfs.conf，添加虚拟主机

  ```less
  server {
       listen 80;
       server_name 192.168.101.65;
       location /group1/M00/{
       	root /home/yuqing/fdfs_storage/data;
       	ngx_FastDFS_module;
       }
  }
  ```

  
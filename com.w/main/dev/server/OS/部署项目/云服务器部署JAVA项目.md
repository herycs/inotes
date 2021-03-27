#          云服务器部署JAVA项目

> 此类博客很多了，但想了想还是写一下记录一下吧

# 1.前置条件

> ​	以下挑选自己需要的使用，并不一定需要全部下载

## 1.1 下载对应软件

> windows下，下载LINUX软件至本地

- JDK：建议去[JDK官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载对应的Liunx系统版本的,tar.gz。

  > 登录你的服务器，可以使用Xshell绑定你的服务器，并登录。如果存在登录不了的，可能是你的服务器的端口的访问限制，此处提一下（腾讯云的服务器登录端口是22默认是放通的，可以去登录腾讯云检查下安全组设置）
  >
  > - 登录后查看你的liunx位数,并下载对应版本的。
  >
  >   ```sql
  >   getconfig LONG_BIT
  >   ```

- Tomcat：建议同样去[JDK官网](https://tomcat.apache.org/download-90.cgi)下载，tar.gz。

  > 选择 Deployer: 下的tar.gz类型下载即可。

- MySQL：官网下载，tar包。

- nginx：[官网下载](http://nginx.org/),tar.gz。

## 1.2 上传软件至服务器

- 安装Xftp软件，并和服务器端口绑定（如有不明白的操作的，请查询相关资料）

- 安装完后连接服务器，看到服务器的文件目录即证明连接成功

- 服务器添加相关目录，用以存放软件包及部署软件包（此处只以JDK为例讲了上传方式，若有多个文件可一次将文件夹都创建完）

  - 建议存放在usr下

    ```mysql
    -- 回退到根路径
    cd /
    -- 切换到local目录
    cd /usr/local
    -- 创建jdk目录用以存放JDK
    mkdir jdk
    ```

    

- 选择好要存到方服务器的具体文件夹

  由于之前已经创建好了，故，选择对于的jdk文件夹，并选择本地数据包，上传

- 其它软件同理

# 2.安装并配置相关环境及软件

## 2.1 JDK配置

> 上传jdk压缩包，方法见1.2节

- 1中我们已经下载了对应的软件，接下来将安装并配置JDK环境，默认你以按照上面的描述上传了文件。

  ```mysql
  -- 切花到指定位置（存放压缩包的位置）
  cd /
  cd /usr/local/jdk
  -- 解压压缩包
  tar -zxvf {jdk-XXX,此处写你的jdk，一般tab键就能自动补全} -C ./jdk
  -- 配置环境变量
  vim /etc/profile
  (进入后将光标调整到文件末尾，按i键进入编辑模式[左下角有insert即为进入编辑模式])
  	-- 文档末尾加上以下配置
  #set JDK environment
  JAVA_HOME=/usr/local/base/java/jdk1.8.0_171
  CLASSPATH=.:$JAVA_HOME/lib.tools.jar
  PATH=$JAVA_HOME/bin:$PATH
  export JAVA_HOME CLASSPATH PATH
  	-- 保存并退出
  	esc键退出insert模式，具体表现是左下角insert消失
  	shift+:两个键进入命令模式，具体表现是左下角出现冒号
  	输入wq，保存并退出，执行成功的表现是退回到命令行
  ```

- 其它操作

  - 验证

    分别命令行输入，java , javac , javac -version查看显示同windows

  - 卸载已安装jdk

    ```mysql
    -- 查询已安装的
    rpm -qa| grep java
    -- 卸载已安装的
    rpm -e --nodeps [此处跟你查询出来的java-XXX]
    ```
    
  - 出现问题
  
      java：error while loading shared libraries: libjli.so: cannot open shared object file: No such file or directory
  
      ```shell
      rm -rf /usr/bin/javac \
      rm -rf /usr/bin/jar \
      rm -rf /usr/bin/java
      
      mkdir /usr/bin/java \
      mkdir /usr/bin/javac \
      mkdir /usr/bin/jar
      
      ln -s /usr/local/jdk/jdk1.8.0_171/bin/java /usr/bin/java \
      ln -s /usr/local/jdk/jdk1.8.0_171/bin/javac /usr/bin/javac \
      ln -s /usr/local/jdk/jdk1.8.0_171/bin/jar /usr/bin/jar
      
      查找libjli.so文件
  find / -name libjli.so
      ldd /usr/bin/java
      mv /usr/local/jdk1.6.0_13/jre/lib/i386/jli/libjli.so /lib
      ```
      

## 2.2 安装mysql

> 上传mysql压缩包，方法见1.2节

- mysql安装

  > mysql是tar文件（打包文件，解压时命令参数使用 -xvf 即可）

  ```mysql
  -- 切花到指定位置（存放压缩包的位置）
  cd /
  cd /usr/local/mysql
  
  -- 解压压缩包
  tar -xvf {mysql-XXX,此处写你的mysql，一般tab键就能自动补全} -C ./mysql
  
  -- 安装客户端及服务端，（可查看解压的文件，ls 即可）
  rpm -ivh mysql {-Server-XXX,此处为你的server服务端，使用tab补全即可}
  
  -- @ 安装完后会提示生成了随机密码，使用下面命令查看即可查看
  cat /root/.mysql_secret
  -- 结果：
  [root@VM_0_12_centos mysql]# cat /root/.mysql_secret 
  
  # The random password set for the root user at Wed May 27 19:51:15 2020 (local time): _CStjbHvLJbsXM6g
  rpm -ivh {mysql-client-XXX,此处为你的mysql服务端，tab补全即可}
  
  -- 启动mysql服务
  service mysql start
  -- 结果
  [root@VM_0_12_centos mysql]# service mysql start
  Starting MySQL.. SUCCESS! 
  
  -- 登录，此处要用到之前的随机密码，在 @ 处
  mysql -u root -p
  {输入密码}
  -- 设置新密码
  set password=password('你的密码');
  
  _CStjbHvLJbsXM6g
  ```

- 本地图形界面操作数据库

  - mysql workbench

    ```mysql
    -- 授权远程登录
    grant all privileges on *.* to 'root'@'%' identified by '密码';
    -- 关闭防火墙或者打开固定端口访问，初次使用可以直接关闭防火墙
        -- 打开3306端口
        /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
        /etc/rc.d/init.d/iptables save
        /etc/init.d/iptables status
        -- 关闭防火墙
        service iptables stop 
    -- 创建连接
    打开mysqlworkbench，新建连接，输入IP及端口，输入用户名，密码
    ```

  - 卸载Mysql同JDK

    ```mysql
    -- 查询已安装的
    rpm -qa| grep mysql
    -- 卸载已安装的
    rpm -e --nodeps [此处跟你查询出来的mysql-XXX]
    ```
  
- 切记若你确实授权了，服务器端口也放通了，（腾讯云）端口也放通了，但是在远程连接的时候就是告诉你10060，连接失败

    - 这时不妨看一下你的云端口是不是写的（::::/0, 3306)，如果是的话不妨改成（0.0.0.0/0，3306)
    - 就为个这，，，我重装了4次

## 2.3安装tomcat

> 上传tomcat压缩包，方法见1.2节

- 安装

  ```mysql
  -- 切花到指定位置（存放压缩包的位置）
  cd /
  cd /usr/local/tomcat
  -- 解压压缩包
  tar -zxvf {tomcat-XXX,此处写你的tomcat，一般tab键就能自动补全} -C ./tomcat
  -- 启动mysql服务
  cd /{apache-XXX,此处是你解压的目录，tab补全即可}/bin
  ./startup.sh
  -- 使用IP访问下，端口8080，如果失败则可能是防火墙的原因，此时去打开对应端口再次尝试即可，成功场景是看见tomcat界面（那只神奇的cat）
  {你的IP}:8080
  ```

## 2.4 安装redis

> redis安装是安装源码的方式，由于其是C语言开发我们需要先部署C环境，在编译安装

- 安装GCC

  ```mysql
  -- 创建文件夹并切换至文件夹
  cd /usr/local
  mkdir c
  cd c
  -- 安装C++环境
  yum install gcc-c++
  	-- {将出现以下提示语句询问是否需下载，一路输入y即可}
  	-- Total download size:20M
  	-- is that oK?
  	y
  	-- From XXX...
  	-- is that OK？
  	y
  ```

- 安装redis

  ​       

  ```sql
  -- 切换目录
  cd /usr/local
  mkdir redis
  cd redis
  -- 下载，这里提供一个版本号的安装
  wget http://download.redis.io/releases/redis-3.0.4.tar.gz  
  -- 解压
  tar -xzvf redis-3.0.4.tar.gz
  -- 切换目录
  cd redis-3.0.4
  -- 编译
  make
  -- 安装至指定目录
  make PREFIX=/usr/local/redis install
  ```

  - 配置

  ```mysql
  -- local下找到redis-3.0.4的目录
  cd redis-3.0.4
  -- 粘贴配置文件至我们的redis编译目录
  cp ./redis.conf /usr/local/redis/bin
  
  -- 启动redis服务端
  ./redis-server redis-conf
  
  -- 启动redis客户端,成功场景是命令行提示符由root@XXX变为ip:port
  ./redis-cli
  ```

## 2.5 安装nginx

- 安装

  - 环境准备

  ```sql
  -- 1.GCC 
  yum install gcc-c++
  -- 2.PCRE
  -- 第三方开发包PCRE，nginx 的 http 模块使用 pcre 来解析正则表达式。
  yum install -y pcre pcre-devel
  -- 3.zlib
  -- nginx 使用 zlib 对 http 包的内容进行 gzip。
  yum install -y zlib zlib-devel
  -- 4.OpenSSL
  -- OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。nginx 不仅支持 http 协议，还支持 https（即在 ssl 协议上传输 http）
  yum install -y openssl openssl-devel
  ```

  - 安装 nginx

  > 需提前将数据包上传至服务器，具体操作见1.2节。

  

  ```mysql
  -- 创建文件夹并进入
  cd /usr/local
  mkdir nginx
  cd nginx
  -- 解压
  tar -zxvf nginx-1.8.0.tar.gz -C ./nginx
  -- 进入解压目录
  cd nginx-1.8.0
  -- 键入以下命令，创建makefile文件
  	-- 下面的位置需要写成你的nginx安装路径
  	-- 下面配置的是临时文件目录
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
      --http-scgi-temp-path=/var/temp/nginx/scgi
  -- 安装
  make install
  ```

- 启动

  ```mysql
  -- 创建临时文件目录，这在makefile中配置了
  mkdir /var/temp/nginx/client -p
  -- 切换路径
  cd /usr/local/ngiux/sbin
  -- 启动
  ./nginx
  -- 可查看相关端口情况，判断是否启动成功
  ps aux|grep nginx
  ```

- 相关命令

  

  ```sql
  -- 关闭
  ./nginx -s stop
  ./nginx -s quit
  -- 重启
  ./nginx -s reload
  ```

- 拓展知识

  - 有关makefile

    - 概述：

      Makefile是一种配置文件， Makefile 一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，其中也可以执行操作系统的命令。

    - 参数及其解释

      ```mysql
      configure参数
      ./configure \
      -- 指向安装目录
      --prefix=/usr \
      -- 指向（执行）程序文件（nginx）
      --sbin-path=/usr/sbin/nginx \
      -- 指向配置文件
      --conf-path=/etc/nginx/nginx.conf \
      -- 指向log
      --error-log-path=/var/log/nginx/error.log \
      -- 指向http-log
      --http-log-path=/var/log/nginx/access.log \ 
      -- 指向pid
      --pid-path=/var/run/nginx/nginx.pid \
      -- （安装文件锁定，防止安装文件被别人利用，或自己误操作。）
      --lock-path=/var/lock/nginx.lock \                   
      --user=nginx \
      --group=nginx \
      -- 启用ngx_http_ssl_module支持（使支持https请求，需已安装openssl）
      --with-http_ssl_module \
      -- 启用ngx_http_flv_module支持（提供寻求内存使用基于时间的偏移量文件）
      --with-http_flv_module \
      -- 启用ngx_http_stub_status_module支持（获取nginx自上次启动以来的工作状态）
      --with-http_stub_status_module \
      -- 启用ngx_http_gzip_static_module支持（在线实时压缩输出数据流）
      --with-http_gzip_static_module \
      -- 设定http客户端请求临时文件路径
      --http-client-body-temp-path=/var/tmp/nginx/client/ \
      -- 设定http代理临时文件路径
      --http-proxy-temp-path=/var/tmp/nginx/proxy/ \
      -- 设定http fastcgi临时文件路径
      --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ \
      -- 设定http uwsgi临时文件路径
      --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
      -- 设定http scgi临时文件路径
      --http-scgi-temp-path=/var/tmp/nginx/scgi \
      -- 启用pcre库
      --with-pcre
      ```

      

# 3.nginx

## 3.1 nginx 静态资源部署

- nginx 静态资源

  ```sql
  -- 上传资源至nginx目录下projects，可自定义目录
  cd /usr/loacl/nginx
  mkdir projects
  ```

- 修改配置文件, nginx/conf/conf.xml

  ```sql
  server {
  	 # 监听的端口
       listen       81;
       server_name  {域名或IP};
       # 访问路径配置
       location / {
       	# 根目录
       	root   projects;
       	# 默认首页
       	index  index.html;
       }
       # 错误页面
       error_page   500 /50x.html;
       location = /50x.html {
      	 root   html;
       }
  }
  ```

- **至此便可浏览器输入IP或域名尝试查看你的项目了。**

## 3.2 nginx 反向代理

- 配置

  ```sql
  upstream proxy_IP{
  	# 代理IP
  	server [IP:port];
  }
  
  server {
      listen       80; # 监听的端口
      server_name  [域名或IP];
      location / {	# 访问路径配置
      	# root   index;# 根目录
      	# proxy_IP是上面配置的IP
      	proxy_pass http://proxy_IP;
      	#  默认首页
      	index  index.html index.htm;
      }
  }
  
  ```

## 3.3 负载均衡

- 前置条件

  需要提前安装两个及以上的Tomcat用作测试，并将端口号修改为不一致，默认为8080，可以改为8081,8082等等。

- 负载均衡配置

  ```sql
    # 配置服务器
    upstream proxy_IP {
  	   server IP:8080;
  	   server IP:8081;
  	   # 可在server IP:port后加比重以配置其分担业务的能力，例如：server IP:8080 weight = 2;
  	   server IP:8082;
      }
  
      server {
          # 监听的端口
          listen       80;
          
          # 域名或ip
          server_name  [域名或IP];
          
          # 访问路径配置
          location / {
          	# root   index;# 根目录
          	# 代理路径
  	    	proxy_pass http://proxy_IP;
          	    index  index.html; # 默认首页
          	}
          	
          # 错误页面
          error_page   500  /50x.html;
          
          location = /50x.html {
              root   html;
          }
      }
  ```

  ​	
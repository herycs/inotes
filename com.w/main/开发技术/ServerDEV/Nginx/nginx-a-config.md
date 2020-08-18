# nginx-config

## 认知

单位：

nginx中配置是否可用指定单位取决于模块的实现

配置中使用变量：少数模块支持

## 配置框架

- 全局配置：服务器用户组，pid路径，日志路径等等
- enents：连接，配置服务器对外提供的连接信息，事件驱动等
- http：
    - server：虚拟主机相关参数
    - location：配置请求的路由，处理各种页面情况

```shell
...              #全局块
events { ... }         #events块
http
{
	...   #http全局块
    server        #server块
    {  
    	...       #server全局块
        location [PATTERN] { ... }  #locations
    }
}
```

## 模板

```shell

# user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;
}


http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    
    error_page 404 https://www.baidu.com; #错误页

    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

## 负载均衡

### 热备

```shell
upstream mysvr {   
	server 127.0.0.1:7878;
	server 192.168.10.121:3333 backup;  #热备
}
```

### 轮询

```shell
upstream mysvr { 
    server 127.0.0.1:7878;
    server 192.168.10.121:3333;       
}
```

### 加权轮询

```shell
upstream mysvr { 
    server 127.0.0.1:7878 weight=1 max_fails=2 fail_timeout=1;
    server 192.168.10.121:3333 weight=2 max_fails=2 fail_timeout=1;
}
```

### ipHash

```shell
upstream mysvr { 
    server 127.0.0.1:7878; 
    server 192.168.10.121:3333;
    ip_hash;
}
```

## Nginx服务的基本配置

### 用于调试进程和定位问题

- 是否以守护方式运行：daemon {on | off}
- 是否以master/worker方式工作：master_process {on | off}
- error日志：error_log [/path/file] [level]
- 是否处理特殊调试点：debug_points [stop | abort]
- 仅仅对指定客户端输出debug日志：debug_connection [IP | CIDR]
- 限制coredump核心转储文件大小：worker_rlimit_core size
- 指定coredump文件生成目录：working_driectory path

### 正常运行的配置项

定义环境变量

嵌入其他配置文件

pid文件路径

Nginx worker进程及用户组

指定Nginx可打开的最大句柄描述符个数

限制信号队列

### 优化性能

worker进程个数

worker绑定到CPU

SSL硬件加速

系统调用gettimeofday执行频率

worker进程优先级

### 事件类配置项

是否打开accept锁

lock文件路径

使用accept锁到建立连接之间的延迟时间

批量建立新链接

选择事件模型

每个worker最大链接数

## 实操静态资源服务

### 虚拟主机与转发

监听端口：listen [address] : [port]

主机名称：server_name name

### 文件路径定义

以root方式设置资源路径

以alias方式设置资源路径

访问首页

依据Http返回码重定向页面

是否允许递归使用error_page

try_files

### 内存、磁盘资源分配

Http包体只存储到磁盘文件

Http包体尽量写入到一个内存buffer中

存储Http头部内存buffer的大小

存储超大Http头部的内存buffer的大小

存储Http包体的内存buffer大小

Http包体临时存放目录

### 网络连接的设置

读取 Http头部超时时间

读取Http包体的超时时间

发送响应的超时时间

### MIME类型的设置

### 限制客户端

按Http方法名限制用户请求

Http请求包体的最大值

对请求的限速

### 文件操作的优化

sendfile系统调用

AIO系统调用

### 客户端请求特殊处理

忽略不合法Http

Http头部是否允许下划线

对If-Modified-Since头部处理策略

## 负载均衡

### upstream块

### server

### ip_hash

### 记录日志时支持的变量

## 反向代理

### proxy_pass

### proxy_method

### proxy_hide_header

### proxy_pass_header

### proxy_pass_request_body

### proxy_pass_request_headers

### proxy_redirect

### proxy_next_upstream


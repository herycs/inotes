# MogoDB简单操作

# 1.安装

# 2.启动

- cmd键入mongodb --dbpath d:\mongodb\data\db

> 辅助软件：studio3t

# 3.创建用户

- cmd键入mongo.exe登录

# 4.配置文件

-  windows下

  - 在MongoDB下建立文件夹：data，logs
  - 建立文件：mongo.conf，logs/mongo.log

- linux下

  - mongo.conf在etc文件夹下

- 配置文件基本：

  ```tex
  #数据库数据存放目录
  dbpath=/usr/local/mongodb304/data
  #数据库日志存放目录
  logpath=/usr/local/mongodb304/logs/mongodb.log 
  #以追加的方式记录日志
  logappend=true
  #端口号 默认为27017
  port=27017 
  #以后台方式运行进程
  fork=true 
  #开启用户认证
  auth=true
  #关闭http接口，默认关闭http端口访问
  nohttpinterface=true
  #mongodb所绑定的ip地址
  bind_ip = 127.0.0.1 
  #启用日志文件，默认启用
  journal=true 
  ```


# 5.将Mongo设置为win服务

- 前置条件
  - 在data文件夹下新建一个log文件夹，用于存放日志文件，在log文件夹下新建文件mongodb.log
  - 在 D:\mongodb文件夹下新建文件mongo.config，并用记事本打开mongo.config输入以下内容:
    - dbpath=D:\mongodb\data\db 
    - logpath=D:\mongodb\data\log\mongodb.log

- 设置服务
  - 以管理员身份打开cmd命令框（开始——输入cmd找到cmd.exe——右键——以管理员身份运行）
  - 进入bin文件（配置环境变量后可直接在任意位置键入此命令）夹输入以下命令mongod --config D:\mongodb\mongo.config --install --serviceName "MongoDB" --journal  
  - 右键计算机——系统服务，打开win7服务框，本地服务服务列表中会看到MongoDB
  - 在cmd中输入 net start mongoDB  开启服务，重新查看win服务窗口，发现mongodb的状态栏显示“已启动”
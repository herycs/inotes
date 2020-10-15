# 设计

## 页面设计

### 我的

<img src="D:\Wp\wpNote\com.w\main\系统设计\毕设\我的模块页设计.png" style="zoom:50%;" />

### 动态

<img src="D:\Wp\wpNote\com.w\main\系统设计\毕设\消息页设计.png" style="zoom:50%;" />

### 发现

<img src="D:\Wp\wpNote\com.w\main\系统设计\毕设\频道页设计.png" style="zoom:50%;" />

### 首页

<img src="D:\Wp\wpNote\com.w\main\系统设计\毕设\首页设计.png" alt="首页设计" style="zoom:50%;" />

帖子详情页

<img src="D:\Wp\wpNote\com.w\main\系统设计\毕设\贴子详情页设计.png" style="zoom:50%;" />

## 数据模型设计

### 表设计

#### 用户

用户信息表

角色表

权限表

好友表（好友，黑名单）

地理位置表

#### 记录

聊天记录

评论

浏览历史

日志表

用户设置

系统设置

#### 基础模块表

页面模块

频道模块

#### 内容

帖子表

#### 文件表

用户头像

用户隐私文件空间

资源表

### 数据库选取

对于关系类型数据表基于MySQL进行存储

对于非关系型数据库基于MongoDB存储

对于少量不持久或者访问量高发的数据基于Redis存储

### 其他技术选择

搜索：使用ES + LogStash

扫描：模块技术采用二维码技术

文件存储：视频，文件，大量图片，采用HDFS存储

数据分析：采用Hadoop进行存储及挖掘

用户画像确认：采用数据挖掘算法进行初步的用户画像构建

数据通信：采用Http协议，同时API对接采用Restful风格

消息通讯：采用RabbitMQ

系统高可用：采用Zookeeper进行构建

部署：采用Docker

一键式管理采用：Jenkins

## 服务端设计

### 架构设计

前后端分离

#### 前端

Vue

#### 后端

Spring + Spring Boot + Spring Cloud

### 业务拆分

#### 用户模块

负责基础的用户信息（头像，昵称，账号，密码，关联手机号，关联邮箱）

其他信息：用户详情，用户偏好，用户个性化设置

#### 好友模块

好友关系

粉丝

关注

聊天记录

#### 记录

浏览历史

日志记录

#### 动态模块

发帖

评论回复

#### 推荐

首页推荐

#### 探索

频道整理

领域整合

### 功能设计

#### 文件管理（upload, download, management）

#### 短信收发（sms）

#### 消息服务（MQ_Service）

#### 运维监控（eureka）

#### 路由

#### - 动态管理（Sass）

### API设计

> Restful 风格




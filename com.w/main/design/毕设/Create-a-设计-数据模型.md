

# 设计-数据模型

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

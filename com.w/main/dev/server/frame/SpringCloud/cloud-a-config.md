# Config

远程Git仓库

Config Server

本地Git仓库

## 访问流程

拉取远程---》存入本地git---》请求（远程失败，则返回本地）

## 访问权限

Config Server 访问Git需要HTTP方式的认证（Username&Password)，SSH

## 支持配置

Git

SVN

本地文件存储

需要将spring.profiles.active=native

## 健康检查

## 属性覆盖

Config Server

## 安全保护

物理网络限制，OAuth2等鉴权，Spring Security

## SpringCloud Config 加解密

使用前提，配置中心运行环境中需要安装不限制长度的JCE版本（默认JRE自带的是限制长度的）

## 加密方式

对称加密

非对称加密

## 高可用

### 传统模式

负载均衡-》提供给客户端列表

### 服务模式

将配置中心注册为基础的服务，这样就可以通过服务名获取服务

## 失败快速响应与重试

## 动态刷新

### 连接GIt

git的推送Web  Hook

### SpringCloud Bus


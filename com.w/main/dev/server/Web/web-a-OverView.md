# Web-Overview

## 架构原则

- 资源由URL表示
- 基于Http与Server进行交互
- 数据展示在浏览器上进行

## DNS过程

1. 浏览器缓存 域名-》IP
2. OS缓存，host文件
3. 向LDNS发起解析请求，一般都是最近的解析DNS或者互联网运营提供商
4. root Server，根域名服务器--->返回LDNS gTLD
5. gTLD Server，主域名服务器--->Name Server
6. Name Server，---》LDNS解析IP + TTL
7. 解析结果给用户---》缓存到本地系统缓存中

其实不见得就只有十步，有的还有负载均衡可能多余10个

JVM中也有域名缓冲（lib/security/java.security）

正确域名缓冲 | 错误域名缓冲

## 域名解析方式

A，MX，CNAME，NS，TXT

A: 域名-》IP

MX：将域名下的邮件服务器指向自己的Mail Server

CNAME：别名解析

NS：为域名指定DNS解析服务器

TXT：为主机域名设置说明

## CDN

内容分布网络，流量分配网络

- 可扩展
- 安全性
- 可靠性，响应和执行

### 负载均衡

集群负载均衡

操作系统负载均衡，利用中断

### CDN动态加速

利用动态链路探测寻找回源的路径

## CMD

域名解析结果：nslookup
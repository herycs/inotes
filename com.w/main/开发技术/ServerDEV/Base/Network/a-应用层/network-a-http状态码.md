# HTTP报文内的信息

##  HTTP报文

构成：

- 多行字符串构成的字符串文本（换行由回车符CR（16进制0x0d）+ 换行符LF(16进制0x0a)）
- 报文大致分为：报文首部+报文主体

图示：

- 请求报文结构

- 响应报文结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191214151040318.png)

- 请求行：请求的方法 + 请求URI + HTTP版本

- 状态行：响应结果状态码 + 原因短语 + HTTP版本

- 首部字段：请求、响应的各种条件和属性的各类首部

- 其它：可能含有HTTP中未定义的首部（Cookie等）


## 编码提升传输效率

## 报文主体和实体

报文：HTTP通信中的基本单位， 8位组字节流组成

实体

- 作用：作为请求或响应的载荷数据
- 内容：实体首部+实体主体

## 压缩内容

HTTP中的内容编码的功能可进行类似压缩内容的操作

类型：

- gzip(Group zip)
- compress(UNIX系统标准压缩)
- deflate(zlib)
- identity(不编码)

## 分割发送（CHunked Transfer）

- 数据分割成多块可实现浏览器内容的部分显示
- 把实体主体分块的功能称为分块传输编码
- 每块使用16进制编码，实体主体的最后一块采用0（CR+LF）来标记

## 多种数据的对象集合

采用了MIME（Multipurpose Internet Mail Extensions）机制

内容：

- multipart/form-data
    - web表单上传文件使用
- multipart/byteranges
    - 状态码206响应报文包含了多个范围的内容时使用

格式：

- boundary字符串划分多部分对象集合指明的各类实体
- 实体起始行前加入--
- 多部分集合最后加入--

- 多部分集合的每个部分类型中都包含首部字段，另外各个部分中嵌套使用多部分对象集合‘

## 获取部分内容的范围请求

首部字段指定资源的byte范围

- 5001~1000字节

```http
Range: bytes=5001-10000
```

- 5001之后全部

```http
Range: bytes=5001-
```

多重范围：0-2000 ， 4000-6000

- 返回状态码：306
- 响应首部字段：Content-Type标明multipart/byteranges

```http
Range: bytes=0-2000, 4000-6000
```

## 内容协商返回最合适的内容

相同URI返回不同界面，机制称为内容协商（Content Negotiation）

判断标准：

- Accept
- Accept-Charset
- Accept-Encoding
- Accept-Language
- Content-Language

服务器驱动协商（Server-driven Negotiation）

- 由服务端进行内容协商。
- 请求首部字段为参考，服务器端自动处理

客户端驱动协商（Agent-driven Negotiation）

- 客户端进行内容协商

透明协商（Transparent Negotiation）

- 由服务端和客户端各自进行内容协商

## 状态码

### 概述

- 返回状态码分三位：状态类别+两位无类别码

- 状态码列表

    | 状态码 | 类别                             | 原因短语                 |
    | ------ | -------------------------------- | ------------------------ |
    | 1XX    | informational（信息性状态码）    | 接收的请求正在处理       |
    | 2XX    | Success（成功状态码）            | 请求正常处理完毕         |
    | 3XX    | Redirection（重定向状态码）      | 需要进行附加操作完成请求 |
    | 4XX    | Client Error（客户端错误码）     | 服务器无法处理请求       |
    | 5XX    | Server Error（服务器错误状态码） | 服务器处理请求出错       |

### 常用状态码

#### 1xx

> 100 -> 103

 [100 Continue](https://baike.baidu.com/item/HTTP状态码/5053660#1_1)

 [101 Switching Protocols](https://baike.baidu.com/item/HTTP状态码/5053660#1_2)

[102 Processing](https://baike.baidu.com/item/HTTP状态码/5053660#1_3)

#### 2XX 成功

> 200 -> 207

200 OK

- 客户端的请求在服务器端被正常处理了
- 在响应报文内随状态码一起返回的信息会因方法的不同而发生改变

204 No Content

- 服务器接收的请求已成功处理，但在返回的报文中不含实体的主体部分，不允许返回任何实体
- 浏览器页面不更新

206 Partial Content

- 客户端发起范围请求，服务端指行了这次GET请求
- 响应报文中包含由Conent-Range指定的范围实体内容

#### 3XX  重定向

> 300 -> 307
>
> 浏览器需要执行某些特殊处理以正确处理请求
>
> 返回码为：301,302,303时几乎所有浏览器会将POST改为GET，并删除请求报文内的主体，之后再次发送
>
> 301，302标准禁止将POST改为GET，但使用时会这么做

301 Moved Permanently

- 永久性重定向
- 所请求资源已被分配了新的URI，以后应该使用资源现在所指的URI

302 Not Found

- 临时性重定向
- 所请求资源已被分配了新的URI，希望本次用户能使用新的URI访问
- 表示资源不是被永久移动，而是临时性移动，将来还可能再变化

303 See Other

- 请求对应的资源存在着另外一个URI，应使用GET定向获取请求的资源

304 Not Modified

- 客户端发送附带条件的请求，服务器端允许请求访问资源，条件不满足
- 服务端资源未改变，可直接使用客户端未过期的缓存
- 状态码返回时，不含任何响应主体部分

307 Temporary Redirect

- 临时重定向
- 307会遵照浏览器协议，不会从POST变成GET处理响应是的行为，每种浏览器出现情况不同

#### 4XX 客户端错误

> 400 -> 451
>
> 客户端是发生错误所在

400 Bad Request

- 请求报文中存在语法错误

401 Unauthorized

- 发送的请求需要有HTTP认证（BASIC认证，DIGEST认证）的认证信息
- 此前已进行过一次请求则表示用户认证失败
- 响应需要包含一个适用于被请求资源的WWW-Authenticate首部用以质询用户信息
- 浏览器初次接收到401会弹出认证窗口

403 Forbidden

- 请求资源的访问被拒绝了
- 服务端没有必要给出详细理由
    - 需要显示理由，可在主体中添加东西
- 不具有文件系统的访问权限也可能是发生403的原因

404 Not Found

- 服务器上没有请求的资源
- 服务器拒绝请求，且不打算说明理由时使用

#### 5XX 服务器错误

> 500 -> 510

500 Internal Server Error

- 服务端在执行请求时发生了错误

503 Service Unavailable

- 服务器暂时处于超负荷或者正在停机维护中，无法处理请求
- 若知道解除以上状况需要的时间可以写入Retry-After字段

504 Gateway Time-out

- 网关超时，上游服务器未在规定时间

505 

- http版本不支持
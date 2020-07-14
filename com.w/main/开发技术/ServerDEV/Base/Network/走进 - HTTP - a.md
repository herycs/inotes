# 走进 - Http - a



# 发展历史

- HTTP/0.9（问世）：1990年，
- HTTP/1.0：1996年5月，记载于RFC1945
    - RFC1945 - Hypertext Transfer Protocol -- HTTP/1.0
- HTTP/1.1：1997年1月，标准RFC2068，修订RFC2616
    - RFC2616 - Hypertext Transfer Protocol -- HTTP/1.1

# TCP/IP

TCP/IP是互联网相关各类协议族的总称

- 协议中存在各式各样的内容。
    - 从电缆到规格到IP的选定方法，寻找异地用户的方法、双方建立通信的顺序，以及web页面显示需要处理的步骤，等等。

## TCP/IP协议分层管理

TCP/IP协议族按层次分：

应用层

- 决定向用户提供服务时通信的活动

传输层

- 对应上层应用，提供处于网络连接中的两台**计算机之间的数据传输**
- 两个协议：
    - TCP（Transmission Control Protocol）传输控制协议
    - UDP（User Data Protocol）用户数据报协议

网络层（网络互联层）

- 处理网络中的流动的**数据包**（数据传输的最小单位）
- 规定了通过怎样的路径到达对方计算机，并把数据包传送给对方

链路层（又名数据链路层，网络接口层）

- 处理连接网络的**硬件**部分
    - 操作系统，硬件的设备驱动，NIC（Network Interface Card，网络适配器，即网卡），光纤等物理可见部分（包括连接器等一切传输媒介）

## 通信传输流

​	![image-20191213232633213](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191213232633213.png)

流程

1. 应用层发出请求
2. 传输层（TCP协议）--->分割报文并标号
3. 网络层（IP协议）--->增加作为通信目的地的MAC地址
    - 链路层传送
4. 网络层
5. 传输层
6. 应用层

流程图示：

![image-20191213235257650](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191213235257650.png)

##  IP协议（Internet Protocol）

- 网际协议位于网络层，TCP/IP 中IP就是指网际协议
- 作用：数据包发送给对方
    - 依赖信息：IP地址+MAC地址（Media Access Control Address）
- 网络中实际应用：采用ARP（Address ResolutionProtocel)解析地址的协议，依据通信方的IP地址反差出对应的MAC地址

## TCP协议

传输层，提供可靠的字节流服务（Byte Stream Sercive）

为了方便传输将大块数据分割成以报文段（segment）为单位的数据包进行管理

可靠性：TCP协议能够确认数据最终是否送达到对方

- 三次握手（three-way handshaking)
- 标记：确认信息是否成功到达的：flag - SYN(synchronize)，ACK（acknowlodgement）
- 流程：
    - 发送端--->(带SYN标志的数据包)
    - (带SYN/ACK的数据包)<---接收端接收到后
    - 发送端--->(回传ACK标志的数据包)，至此握手结束

## DNS服务（Domain Name System）

- 应用层
- 域名到IP的解析服务，IP到域名的反查服务

## URI和URL

- URI（统一资源标识符）
- URL（Uniform Resource Locator，统一资源定位符）

## 统一资源标识符

URI（Uniform Resource Identifier）

- Uniform：规定统一的格式可方方便处理多种不同类型的资源，不用根据上下文环境来识别资源指定的访问方式
- Resource：资源的定义是“可标识的任何东西”，可以是单个，也可以是多数集合体
- Identifier：可标识的对象

URI用字符串标识某一互联网资源 > URL表示资源的地点

## URI格式

表示指定的URI，要使用涵盖全部必要信息的绝对URI，绝对URL相对URL

- 相对URL，是指从浏览器中基本URI指定的URL

示例：http://use:pass@www.text.com:8080/dir/doc/index.jsp?id=1#page1

- http://协议信息
- use:pass登录信息（可选）
- www.text.com服务器地址
- 8080服务端口号
- dir/doc/index.jsp带层次的文件路径
- id=1查询字符串（可选）
- page1片段标识符（可选）

# HTTP协议

- 协议规定：请求从客户端发出，服务端响应并返回
- 无状态（stateless）协议（不保存状态，不对请求和响应之间的通信状态进行保存），即就是HTTP这个级别，协议对发送过的请求和响应都不做持久化处理
- 设计之初为了快速处理大量事务，确保协议的可升缩性
    - 现在由于业务需要，需要保存状态等，引入Cookie技术实现这个操作

## 定位资源

使用URI

客户端请求访问资源时，**URI**将作为请求报文中的 **请求URI** 包含在内

- URI 为完整的请求URI
- 例如：GET http://www.baidu.com HTTP/1.1

对服务器本省发起请求（* 代替 请求URI）

- OPTIONS * HTTP/1.1

## 告知服务器意图的HTTP方法

### GET

- 获取资源
- 请求已被URI识别资源，指定资源被服务器解析后返回响应内容
    - 资源为**文本**：保持原样返回
    - 资源为类似CGI（Common Gateway Interface，通用网关接口）的**程序**：返回执行后的输出结果

### POST

- 传输实体主体
- GET方法也可传输实体，但POST主要内容并非获取响应内容

### PUT

- 传输文件
- HTTP/1.1时不带验证机制，存在安全问题
    - 一般使用REST(Representational State Transfer，表征状态转移)标准的同类WEB网站，可能开放使用

### HEAD

- 获取报文首部
- 和GET方法一样，但不返回报文主体
- 一般用于确认URI的有效性及资源更新日期

### DELETE

- 删除文件
- HTTP/1.1时不带验证机制
    - 一般使用REST(Representational State Transfer，表征状态转移)标准的同类WEB网站，可能开放使用

### OPTIONS

- 询问支持的方法
- 查询请求URI支持的方法

### TRACE

- 追踪路径
- 让WEB服务器端将之前请求的通信返回给客户端的方法
- 操作：
    - 发送请求时在Max-Forwards中填入数值每过一层减一（直至到0时停止传输），最后接到请求的服务器返回200 OK 的响应
- 可以查询发送出去的请求是如何被加工的
- 风险：
    - 容易受到（XST， Cross-Site Tracing，跨站追踪的攻击），通常不使用

### CONNECT

- 要求用隧道协议连接代理
- 实现用隧道协议进行TCP通信，主要使用SSL（Secure Sockets Layer，安全套接层)和TSL（Transport Layer Security，传输层安全）协议加密通信内容后经过网络隧道传输

## 1.0和1.1支持的方法

> X---不支持，O---支持

| 协议    | 1.0  | 1.1  |
| ------- | ---- | ---- |
| LINK    | O    | X    |
| UNLINK  | O    | X    |
| GET     | O    | O    |
| POST    | O    | O    |
| PUT     | O    | O    |
| DELETE  | O    | O    |
| OPTIONS | X    | O    |
| TRACE   | X    | O    |
| CONNECT | X    | O    |

## 持久连接

- 初识的HTTP通信要求每次通信都要断开TCP
- 应对策略
    - HTTP Persistent Connections( 别名：HTTP keep-alive或HTTP connection reuse)
- HTTP/1.1中默认使用持久化连接

## 管线化

持久连接使得多数请求以管线化（pipelining)方式发送成为可能，不用等响应即可发送下一个请求

## 使用Cookie

Cookie通过在请求和响应报文中写入Cookie来控制客户端的状态

Cookie根据响应报文中的set-Cookie的首部字段通知客户端保存Cookie

加入Cookie后的请求流程：

- 1.客户端请求：首部不含Cookie
- 2.服务端响应：响应报文中含有通知客户端保存Cookie的字段，客户端保存Cookie
- 3.客户端请求：首部自动添加上保存的Cookie

# HTTP报文内的信息

##  HTTP报文

构成：

- 多行字符串构成的字符串文本（换行由回车符CR（16进制0x0d）+ 换行符LF(16进制0x0a)）
- 报文大致分为：报文首部+报文主体

图示：

- 请求报文结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191214151028968.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

- 响应报文结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191214151040318.png)

- 请求行：

    - 请求的方法 + 请求URI + HTTP版本

- 状态行：

    - 响应结果状态码 + 原因短语 + HTTP版本

- 首部字段：

    - 请求、响应的各种条件和属性的各类首部

- 其它：

    - 可能含有HTTP中未定义的首部（Cookie等）

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

# HTTP返回状态码

## 概述

- 返回状态码分三位：状态类别+两位无类别码

- 状态码列表

    | 状态码 | 类别                             | 原因短语                 |
    | ------ | -------------------------------- | ------------------------ |
    | 1XX    | informational（信息性状态码）    | 接收的请求正在处理       |
    | 2XX    | Success（成功状态码）            | 请求正常处理完毕         |
    | 3XX    | Redirection（重定向状态码）      | 需要进行附加操作完成请求 |
    | 4XX    | Client Error（客户端错误码）     | 服务器无法处理请求       |
    | 5XX    | Server Error（服务器错误状态码） | 服务器处理请求出错       |

## 常用状态码

### 2XX 成功

200 OK

- 客户端的请求在服务器端被正常处理了
- 在响应报文内随状态码一起返回的信息会因方法的不同而发生改变

204 No Content

- 服务器接收的请求已成功处理，但在返回的报文中不含实体的主体部分，不允许返回任何实体
- 浏览器页面不更新

206 Partial Content

- 客户端发起范围请求，服务端指行了这次GET请求
- 响应报文中包含由Conent-Range指定的范围实体内容

### 3XX  重定向

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

### 4XX 客户端错误

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

### 5XX 服务器错误

500 Internal Server Error

- 服务端在执行请求时发生了错误

503 Service Unavailable

- 服务器暂时处于超负荷或者正在停机维护中，无法处理请求
- 若知道解除以上状况需要的时间可以写入Retry-After字段

505 

- http版本不支持

# Web服务器相关概念

## 单虚拟机实现多域名

- HTTP/1.1规范允许一台HTTP服务器搭建多个Web站点（利用了虚拟主机的功能：Virtual Host，虚拟服务器）
- 相同地址下虚拟主机可寄存多个不同主机名和域名的网站，请求时需要在Host首部内完整指定主机名或域名的URI

## 通信数据转发处理程序

> 使用代理可以减少网络带宽的流量，组织内部对特定网站进行访问控制等等

### 代理

- 具有转发功能的**应用程序**
- 不改变请求URI，直接发送给拥有资源的服务器（源服务器）
- 分类基准：
    - 是否使用缓存
    - 是否修改报文

#### 缓存代理(Caching Proxy)

预先将资源的副本保存在代理服务器上

#### 透明代理（Transparent Proxy）

不对报文做任何加工的代理类型

#### 非透明代理

对报文进行加工的代理

### 网关

- 转发其它服务器通信数据的**服务器**
- 可使通信线路上的服务器提供非HTTP协议服务
- 可提高通信安全，可在客户端和网关之间的通信线路上加密以确保连接安全

### 隧道

- 相隔甚远的客户端和服务器两者之间进行**中转**，并保持双方**通信连接**
- 保证客户端和服务端可进行安全的通信，其本身并不会解析HTTP请求
- 隧道会在通信双方断开连接时结束

## 保存资源的缓存

优势避免多次从源服务器转发资源，客户端可以就近从缓存服务器上获取资源

### 缓存有效期

即使存在缓存也会因客户端要求，缓存有效期等因素，确认资源的有效性

### 客户端缓存

客户端一般也会存放部分缓存数据，客户端缓存称为临时网络文件（Trmporary Internet Explorer）

# HTTP首部

## 字段结构

```http
首部字段名: 字段值
```

## 字段类型

> 依据实际用途分

通用首部字段（General Header Fields）

- 请求响应双方都会使用

请求首部字段（Request Header Fields)

- 客户端向服务端发送请求报文时使用的首部， 补充请求附加内容

响应首部字段（Response Header Fields）

- 服务器响应时使用的首部字段，补充响应附加内容

实体首部字段（Entity Header Fields）

- 针对请求报文和响应报文的实体部分使用的首部，补充与实体有关的信息

## HTTP/1.1 首部字段一览

### 通用首部字段

| 首部字段名        | 说明                       |
| :---------------- | :------------------------- |
| Cache-Control     | 控制缓存的行为             |
| Connection        | 逐跳首部、连接的管理       |
| Date              | 创建报文的日期时间         |
| Pragma            | 报文指令                   |
| Trailer           | 报文末端的首部一览         |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议             |
| Via               | 代理服务器的相关信息       |
| Warning           | 错误通知                   |

### 请求首部字段

| 首部字段名          | 说明                                          |
| :------------------ | :-------------------------------------------- |
| Accept              | 用户代理可处理的媒体类型                      |
| Accept-Charset      | 优先的字符集                                  |
| Accept-Encoding     | 优先的内容编码                                |
| Accept-Language     | 优先的语言（自然语言）                        |
| Authorization       | Web认证信息                                   |
| Expect              | 期待服务器的特定行为                          |
| From                | 用户的电子邮箱地址                            |
| Host                | 请求资源所在服务器                            |
| If-Match            | 比较实体标记（ETag）                          |
| If-Modified-Since   | 比较资源的更新时间                            |
| If-None-Match       | 比较实体标记（与 If-Match 相反）              |
| If-Range            | 资源未更新时发送实体 Byte 的范围请求          |
| If-Unmodified-Since | 比较资源的更新时间（与If-Modified-Since相反） |
| Max-Forwards        | 最大传输逐跳数                                |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                |
| Range               | 实体的字节范围请求                            |
| Referer             | 对请求中 URI 的原始获取方                     |
| TE                  | 传输编码的优先级                              |
| User-Agent          | HTTP 客户端程序的信息                         |

### 响应首部字段

| 首部字段名         | 说明                         |
| ------------------ | ---------------------------- |
| Accept-Ranges      | 是否接受字节范围请求         |
| Age                | 推算资源创建经过时间         |
| ETag               | 资源的匹配信息               |
| Location           | 令客户端重定向至指定URI      |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Retry-After        | 对再次发起请求的时机要求     |
| Server             | HTTP服务器的安装信息         |
| Vary               | 代理服务器缓存的管理信息     |
| WWW-Authenticate   | 服务器对客户端的认证信息     |

### 实体首部字段

| 首部字段名       | 说明                         |
| ---------------- | ---------------------------- |
| Allow            | 资源可支持的HTTP方法         |
| Content-Encoding | 实体主体适用的编码方式       |
| Content-Language | 实体主体的自然语言           |
| Content-Length   | 实体主体的大小（单位：字节） |
| Content-Location | 替代对应资源的URI            |
| Content-MD5      | 实体主体的报文摘要           |
| Content-Range    | 实体主体的位置范围           |
| Content-Type     | 实体主体的媒体类型           |
| Expires          | 实体主体过期的日期时间       |
| Last-Modified    | 资源的最后修改日期时间       |

## 非HTTP/1.1首部字段

在HTTP协议通信交互中使用到的首部字段，不限于RFC2616中定义的47中字段，还有Cookie，Set-Cookie和Content-DIsposition等。这些字段归纳在REF4229 HTTP Header Field Registrations中

## End-to-end首部和Ho-by-hop首部

> HTTP首部将定义成缓存代理和非缓存代理的行为分成两种：端到端首部，逐跳首部

端到端首部（End-to-end）

- 此类首部会转发给请求或响应的最终目标
- 要求：
    - 必须保存在由缓存生成的响应中
    - 必须被转发

逐跳首部（Hop-by-hop Header）

- 此类中的首部只对单次转发有效，会因通过缓存或者代理而不再转发
- 若要使用hop-by-hop首部需要提供Connection首部字段
- 详细字段：
    - Connection
    - Keep-Alive
    - Proxy-Authenticate
    - Proxy-Authorization
    - Trailer
    - TE
    - Transfer-Encoding
    - Upgrade

> 除以上字段外都是端到端首部字段

## HTTP/1.1 通用首部字段

请求响应都会用到的首部

### Cache-Control

> [推荐阅览1](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)

缓存请求指令一览

| 指令            | 参数   | 说明                         |
| --------------- | ------ | ---------------------------- |
| no-cache        | 无     | 强制向服务器再次验证         |
| no-store        | 无     | 不缓存请求或响应的任何内容   |
| max-age         | 必须   | 响应的最大Age                |
| max-stale       | 可省略 | 接收已过期的响应             |
| min-fresh       | 必须   | 期望在指定时间内的响应仍有效 |
| no-transform    | 无     | 代理不可更改媒体类型         |
| only-if-cache   | 无     | 从缓存获取资源               |
| cache-extension | -      | 新指令标记（token)           |

缓存响应指令一览

| public           | 无     | 可向任意方提供响应的缓存                       |
| ---------------- | ------ | ---------------------------------------------- |
| private          | 可省略 | 仅向特定用户提供响应的缓存                     |
| no-cache         | 可省略 | 缓存前必须先确认其有效性                       |
| no-store         | 无     | 不缓存请求或响应的任何内容                     |
| no-transform     | 无     | 代理不可更改媒体类型                           |
| must-revalidate  | 无     | 可缓存但必须再向源服务器进行确认               |
| proxy-revalidate | 无     | 要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age          | 必需   | 响应的最大Age值                                |
| s-maxage         | 必需   | 公共缓存服务器响应的最大Age值                  |
| cache-extension  | -      | 新指令标记（token)                             |

> cache-extension实现对Cache-Control首部指令的添加，但只有能识别的服务器才有效，否则会被忽略

### Connection

控制不再转发给代理的首部字段

格式：

```
Connection: 不再转发的字段名
```

示例：

- 代码：

    ```html
    Upgrade: HTTP/1.1
    Connection: Upgrade
    ```


效果：

- 客户端发请求--->服务端
- 服务端响应（将Upgrade删除）--->客户端

管理持久连接

- 格式：

    ```html
    服务端想断开连接时设置字段：Connection: close
    
    服务端想保持连接时: Connection: Keep-Alive
    ```

### Date

表明报文创建日期

格式：

- RFC1123规定格式：

    ```html
    Date: Tue, 04 Jul 2019 04:30:34 GMT
    ```

- 另外一种格式：

    ```html
    Date：Tue Jul 04 04:30:34 2019
    ```

### Pragma

- 历史遗留字段，仅作为与HTTP/1.0向后兼容定义

- 请求转发过程中的所有中间服务器不缓存，Pragma：no-cache

- 所有中间服务器都支持HTTP/1.1则可直接使用Cache-Control: no-cache

- 若不确定，为保险起见使用：

    ```http
    Cache-Control: no-cache
    Pragma: no-cache
    ```

### Trailer

会说明在报文主体后记录了哪些首部字段，该首部字段可应用在HTTP/1.1版本分块传输编码时

### Transfer-Encoding

规定传输报文主体时的编码方式

### Upgrade

指定一个完全不同的通信协议

- 使用Upgrade，需要同时使用：Connection: Upgrade

### Via

为了追踪客户端与服务器之间的请求和响应报文的传输路径，报文经过代理或网关时会在首部Via中附加该服务器的信息，再进行转发

- 作用:
    - 追踪报文转发的路径
    - 避免请求回环的发生

> 通常和TRACE 方法一起使用

### Warning

演变来源HTTP/1.0的响应首部（Retry-After)

通常会告知用户一些与缓存相关的问题的警告

- 格式：

    ```html
    Warning: [警告码][警告的主机:端口号]"[警告内容]"（[日期时间]）
    ```

## 请求首部字段

### Accept

> 媒体类型添加优先级：
>
> 使用q=表示权重（值越大优先级越高），分号进行分割，范围（0-1），精度：小数点后三位

用户代理能够处理的媒体类型及媒体类型的相对优先级

可使用：type/subtype一次指定多种媒体类型

- 文本文件
- 图片文件
- 视频文件
- 应用程序使用的二进制文件

###  Accept-Charset

告知服务器用户代理支持的字符集及字符集的相对优先顺序，可一次指定多个，q为权重

### Accept-Encoding

告知服务器用户代理支持的编码内容及内容编码的优先顺序，可一次指定多个，q为权重

可使用 * 作为通配符，指定任意编码格式

### Accept-Language

告知服务器用户代理能够处理的自然语言集，及自然语言集的相对优先级，可一次指定更多个，q为权重

### Authorization

告知服务器用户代理的认证信息

### Expect

告知服务器期望出现的某种特定行为

服务器无法理解时返回417 Exception Failed

### From

告知服务器使用用户代理的用户电子邮件地址

### Host

告知服务器请求的资源所处的互联网主机名和端口号

### If-Match

> 服务器收到请求后，判断条件为真时执行请求

告知服务器匹配资源所用的实体标记值

- 服务器会比对请求的ETag与资源的ETag值
    - 一致=>执行请求
    - 不一致=>41 Precondition Failed
- 若 If-Match值为 * 则会忽略此值

### If-Modified-Since

若字段值早于资源的更新时间期望处理该请求

- 资源未更新=>304 Not Modified

用来确认资源有效性

- 获取资源的更新时间：通过确认首部字段Last-Modified

### If-None-Match

字段值与资源ETag不一致，告知服务器处理请求

GET或HEAD方法中使用此字段获取最新资源

### If-Range

告知服务器若指定的If-Range字段值与请求资源的ETag值或时间一致，作为范围请求处理，否则，返回全部实体

### If-Unmodified-Since

指定的请求资源只有在字段值内指定的日期之后，未发生更新，则处理请求

### Max-Forwards

TRACE或OPTIONS发送含该字段请求时，该字段以10进制整数形式指定可经过的服务器最大数目，每经过一个减一，为0时直接返回响应，不再转发

### Proxy-Authorization

告知服务器认证所需要的信息

### Range

需求资源的范围

### Referer

> 正确拼写：Referrer

告知服务器请求的原始资源的URI

处于安全性考虑可以不发送该字段

### TE

告知服务器客户端能够处理响应的传输编码方式及相对优先级

可指定trailer字段的分块传输编码方式

- 应用时，将trailers赋值给该字段值

- 例如：

    ```http
    TE: trailers
    ```

### User-Agent

会将创建请求的浏览器和用户代理名称等信息发给服务器

## 响应首部字段

### Accept-Ranges

告知客户端服务端能否处理范围请求

- 能=>指定为bytes
- 不能=>none

### Age

告知客户端源服务器在多久之前创建了响应

单位：秒

### Etag

告知客户端实体标识（可将资源以字符串形式做唯一性标识的方式）

服务器根据每份资源分配对应的ETage值

- 分类：
    - 强Etag:
        - 无论发生多磨细微的变化都会改变其值
    - 弱Etag
        - 只用于提示资源是否相同，只有发生差异时才会改变ETag的值

### Location

将响应接收方引导至某个与请求URI位置不同的资源

- 一般配合3xx ：Redirection的响应，提供重定向的URI

### Proxy-Authenticate

代理服务器要求的认证信息发给客户端

### Retry-After

告知客户端多久之后再发送请求

配合503 Service Unavailable响应或3xx Redirect响应

- 值：
    - 具体时间
    - 创建响应后的秒数

### Server

告知客户端当前服务器上安装的**应用程序**的信息

### Vary

对缓存进行控制

代理服务器接收到源服务器返回的包含Vary指定项的响应后若要再进行缓存，仅对请求中含有相同Vary指定首部字段的请求返回缓存

### WWW-Authenticate

告知客户端适用于请求URI所指定资源的认证方案（Basic或Digest）和带参数提示质询

## 实体首部字段

### Allow

告知客户端请求URI指定资源的所有HTTP方法

- 服务器收到不支持的方法时=>405 Not Allowed

### Content-Encoding

告知客户端服务端对实体的主体部分选用的内容编码方式

### Content-Language

告知客户端主体实体使用的自然语言

### Content-Length

实体主体的大小（字节）

使用内容编码时，不能在使用Content-Length首部字段

### Content-Location

给出报文主体部分相对应的URI

### Content-MD5

检查报文主体在传输过程中是否完整

### Content-Range

当前发送部分及整个实体的大小

- 示例：

    ```html
    Content-Range: bytes 2000-3000/6000
    ```

### Content-Type

主体内对象的媒介的类型

### Expires

告知客户端资源失效日期

当首部字段Cache-Control有指定max-age指令时会优先处理max-age

### Last-Modified

资源最后被修改时间

## 为Cookie服务的首部字段

> 调用Cookie时，由于可校验Cookie的有效期，发送方的域，路径，协议等信息，正规发布的Cookie内的数据不会因来自其他Web站点和攻击者的攻击而泄露（其安全性有待考究）

### Set-Cookie

属性

- expires
    - 指定浏览器发送Cookie的有效期
- path
    - 用于指定Cookie发送范围的文件目录(但有方法可避开)
- domain
    - 可通过domain属性的域名与结尾匹配一致
- secure 
    - 限制Web页面只有在HTTPS情况下才可发送Cookie
    - 当省略secure属性时，不论HTTP还是HTTPS都会对Cookie进行回收
- HttpOnly
    - 使得Javascript无法获取Cookie

### Cookie

- 客户端想获得HTTP状态管理支持时，就会在请求中包含从服务器接收到的Cookie（接收到多个也可发送多个）

## 其它首部字段

### X-Frame-Options

控制网站内容在其他Web网站Frame标签内的显示问题，主要防止点击劫持攻击

- 值：
    - DENY：拒绝
    - SAMEORING：仅同域名下的页面匹配时许可

### X-XSS-Protection

针对跨站脚本攻击，用于控制浏览器XSS防护开关

- 值：
    - 1：XSS过滤设置为无效
    - 0：XSS过滤设置为有效

### DNT

- Do Not Track ：拒绝个人信息采集

### P3P（The Partform for Privacy Preferences）

让个人隐私变成程序可理解的东西，达到保护用户隐私的目的

- 设立步骤：
    - 1.创建P3P隐私
    - 2.创建P3P隐私对照文件后，命名保存在/w3c/p3p.xml
    - 3.从P3P隐私中新建Compact policies输出道HTTP响应中
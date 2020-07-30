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

## TCP/IP协议分层

TCP/IP协议族按层次分：

应用层：决定向用户提供服务时通信的活动

传输层：对应上层应用，提供处于网络连接中的两台**计算机之间的数据传输**

- 两个协议：
    - TCP（Transmission Control Protocol）传输控制协议
    - UDP（User Data Protocol）用户数据报协议

网络层（网络互联层）

- 处理网络中的流动的**数据包**（数据传输的最小单位）
- 规定了通过怎样的路径到达对方计算机，并把数据包传送给对方

链路层（又名数据链路层，网络接口层）：处理连接网络的**硬件**部分

- 操作系统，硬件的设备驱动，NIC（Network Interface Card，网络适配器，即网卡），光纤等物理可见部分（包括连接器等一切传输媒介）

​	![image-20191213232633213](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191213232633213.png)

流程

1. 应用层发出请求
2. 传输层（TCP协议）--->分割报文并标号
3. 网络层（IP协议）--->增加作为通信目的地的MAC地址
    - 链路层传送+物理层处理
4. 网络层
5. 传输层
6. 应用层

流程图示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730144316791.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

##  IP协议（Internet Protocol）

网际协议位于网络层，TCP/IP 中IP就是指网际协议

作用：数据包发送给对方

- 依赖信息：IP地址+MAC地址（Media Access Control Address）

网络中实际应用：采用ARP（Address ResolutionProtocel)解析地址的协议，依据通信方的IP地址反差出对应的MAC地址

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

## 请求方式

### GET

> 获取资源

请求已被URI识别资源，指定资源被服务器解析后返回响应内容
- 资源为**文本**：保持原样返回
- 资源为类似CGI（Common Gateway Interface，通用网关接口）的**程序**：返回执行后的输出结果

### POST

> 传输实体主体

GET方法也可传输实体，但POST主要内容并非获取响应内容

### PUT

> 传输文件

HTTP/1.1时不带验证机制，存在安全问题

- 一般使用REST(Representational State Transfer，表征状态转移)标准的同类WEB网站，可能开放使用

### HEAD

> 获取报文首部

和GET方法一样，但不返回报文主体

一般用于确认URI的有效性及资源更新日期

### DELETE

> 删除文件

HTTP/1.1时不带验证机制

- 一般使用REST(Representational State Transfer，表征状态转移)标准的同类WEB网站，可能开放使用

### OPTIONS

> 询问支持的方法

查询请求URI支持的方法

### TRACE

> 追踪路径

让WEB服务器端将之前请求的通信返回给客户端的方法，可以查询发送出去的请求是如何被加工的

操作：发送请求时在Max-Forwards中填入数值每过一层减一（直至到0时停止传输），最后接到请求的服务器返回200 OK 的响应

风险：容易受到（XST， Cross-Site Tracing，跨站追踪的攻击），通常不使用

### CONNECT

> 要求用隧道协议连接代理

实现用隧道协议进行TCP通信，主要使用SSL（Secure Sockets Layer，安全套接层)和TSL（Transport Layer Security，传输层安全）协议加密通信内容后经过网络隧道传输

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

#### 2XX 成功

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

500 Internal Server Error

- 服务端在执行请求时发生了错误

503 Service Unavailable

- 服务器暂时处于超负荷或者正在停机维护中，无法处理请求
- 若知道解除以上状况需要的时间可以写入Retry-After字段

504 Gateway Time-out

- 网关超时，上游服务器未在规定时间

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

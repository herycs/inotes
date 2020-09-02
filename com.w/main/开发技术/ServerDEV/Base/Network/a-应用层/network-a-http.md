# HTTP协议

## 发展历史

- HTTP/0.9（问世）：1990年，
- HTTP/1.0：1996年5月，记载于RFC1945
    - RFC1945 - Hypertext Transfer Protocol -- HTTP/1.0
- HTTP/1.1：1997年1月，标准RFC2068，修订RFC2616
    - RFC2616 - Hypertext Transfer Protocol -- HTTP/1.1

## 内容

- 协议规定：请求从客户端发出，服务端响应并返回
- 无状态（stateless）协议（不保存状态，不对请求和响应之间的通信状态进行保存），即就是HTTP这个级别，协议对发送过的请求和响应都不做持久化处理
- 设计之初为了快速处理大量事务，确保协议的可升缩性
    - 现在由于业务需要，需要保存状态等，引入Cookie技术实现这个操作

## 定位资源

使用URI

客户端请求访问资源时，**URI**将作为请求报文中的 **请求URI** 包含在内

- URI 为完整的请求URI
- 例如：GET http://www.baidu.com HTTP/1.1

对服务器本身发起请求（* 代替 请求URI）

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
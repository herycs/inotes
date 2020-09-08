# HTTP首部

## 字段结构

```http
首部字段名: 字段值
```

## 字段类型

> 依据实际用途分

通用首部字段（General Header Fields）：请求响应双方都会使用

请求首部字段（Request Header Fields)：客户端向服务端发送请求报文时使用的首部， 补充请求附加内容

响应首部字段（Response Header Fields）：服务器响应时使用的首部字段，补充响应附加内容

实体首部字段（Entity Header Fields）：针对请求报文和响应报文的实体部分使用的首部，补充与实体有关的信息

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
    - 强Etag：无论发生多磨细微的变化都会改变其值
    - 弱Etag：只用于提示资源是否相同，只有发生差异时才会改变ETag的值

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
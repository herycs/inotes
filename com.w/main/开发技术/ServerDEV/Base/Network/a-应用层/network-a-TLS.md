# TLS

Transport Layer Security，安全传输层协议（TLS）用于在两个通信应用程序之间提供保密性和数据完整性。

由来：

HTTPS的推出受到了很多人的欢迎，在SSL更新到3.0时， 互联网工程任务组（IETF）对SSL3.0进行了标准化，并添加了少数机制（但是几乎和SSL3.0无差异），并将其更名为TLS1.0(Transport Layer Security 安全传输层协议)，可以说TLS就是SSL的新版本3.1

1. SSL与TLS两者所使用的算法是不同的

2. TLS增加了许多新的报警代码，比如解密失败(decryption_failed)、记录溢出(record_overflow)、未知CA(unknown_ca)、拒绝访问(access_denied)等，但同时也支持SSL协议上所有的报警代码!

在认证证书时TLS指定必须与TLS之间交换证书， SSL必须与SSL之间交换证书。



该协议由两层组成： TLS 记录协议（TLS Record）和 TLS [握手协议](https://baike.baidu.com/item/握手协议/4058729)（TLS Handshake）。



记录层协议确定传输层数据的封装格式

传输层安全协议使用[X.509](https://baike.baidu.com/item/X.509)认证，之后利用非对称加密演算来对通信方做身份认证，之后交换对称密钥作为会谈密钥（[Session key](https://baike.baidu.com/item/Session key)）。



TLS协议采用[主从式架构](https://baike.baidu.com/item/主从式架构)模型，用于在两个应用程序间透过网络创建起安全的连线，防止在交换数据时受到[窃听](https://baike.baidu.com/item/窃听)及篡改。



TLS协议的优势是与高层的[应用层](https://baike.baidu.com/item/应用层)协议（如[HTTP](https://baike.baidu.com/item/HTTP)、[FTP](https://baike.baidu.com/item/FTP)、[Telnet](https://baike.baidu.com/item/Telnet)等）无耦合。应用层协议能透明地运行在TLS协议之上，由TLS协议进行创建加密通道需要的协商和认证。应用层协议传送的数据在通过TLS协议时都会被加密，从而保证通信的私密性。



### SSL 1.0、2.0和3.0

- 1.0版本从未公开过，因为存在严重的安全漏洞。
- 2.0版本在1995年2月发布，但因为存在数个严重的安全漏洞而被3.0版本替代。
- 3.0版本在1996年发布，是由网景工程师Paul Kocher、Phil Karlton和Alan Freier完全重新设计的。较新版本的SSL/TLS基于SSL 3.0。SSL 3.0作为历史文献[IETF](https://baike.baidu.com/item/IETF)通过RFC 6101发表。

### TLS 1.0

TLS 1.0包括可以降级到SSL 3.0的实现，这削弱了连接的安全性。

### TLS 1.1

TLS 1.1在RFC 4346中定义，于2006年4月发表，它是TLS 1.0的更新。在此版本中的差异包括：

- 添加对CBC攻击的保护：

- - 隐式[IV](https://baike.baidu.com/item/IV)被替换成一个显式的[IV](https://baike.baidu.com/item/IV)。
    - 更改分组密码模式中的填充错误。

- 支持[IANA](https://baike.baidu.com/item/IANA)登记的参数。

### TLS 1.2

TLS 1.2在RFC 5246中定义，于2008年8月发表。它基于更早的TLS 1.1规范。主要区别包括：

- 可使用密码组合选项指定伪随机函数使用SHA-256替换[MD5](https://baike.baidu.com/item/MD5)-[SHA-1](https://baike.baidu.com/item/SHA-1)组合。
- 可使用密码组合选项指定在完成消息的哈希认证中使用SHA-256替换MD5-SHA-1算法，但完成消息中哈希值的长度仍然被截断为96位。
- 在握手期间MD5-SHA-1组合的数字签名被替换为使用单一[Hash](https://baike.baidu.com/item/Hash)方法，默认为SHA-1。
- 增强服务器和客户端指定Hash和签名算法的能力。
- 扩大经过身份验证的加密密码，主要用于GCM和CCM模式的AES加密的支持。
- 添加TLS扩展定义和[AES](https://baike.baidu.com/item/AES)密码组合。所有TLS版本在2011年3月发布的RFC 6176中删除了对SSL的兼容，这样TLS会话将永远无法协商使用的SSL 2.0以避免安全问题。

### TLS 1.3

参见：来回通信延迟

TLS 1.3在RFC 8446中定义，于2018年8月发表。它基于更早的TLS 1.2规范，与TLS 1.2的主要区别包括：

- 将密钥协商和认证算法从密码包中分离出来。
- 移除脆弱和较少使用的命名椭圆曲线支持（参见[椭圆曲线密码学](https://baike.baidu.com/item/椭圆曲线密码学)）。
- 移除MD5和SHA-224[密码散列函数](https://baike.baidu.com/item/密码散列函数)的支持。
- 请求数字签名，即便使用之前的配置。
- 集成HKDF和半短暂DH提议。
- 替换使用[PSK](https://baike.baidu.com/item/PSK)和票据的恢复。
- 支持1-RTT握手并初步支持0-RTT。
- 通过在(EC)DH密钥协议期间使用临时密钥来保证完善的[前向安全性](https://baike.baidu.com/item/前向安全性)。
- 放弃许多不安全或过时特性的支持，包括[数据压缩](https://baike.baidu.com/item/数据压缩)、重新协商、非AEAD密码本、静态[RSA](https://baike.baidu.com/item/RSA)和静态[DH](https://baike.baidu.com/item/DH)密钥交换、自定义[DHE](https://baike.baidu.com/item/DHE)分组、点格式协商、更改密码本规范的协议、UNIX时间的Hello消息，以及长度字段AD输入到AEAD密码本。
- 禁止用于向后兼容性的SSL和RC4协商。
- 集成会话散列的使用。
- 弃用记录层版本号和冻结数以改进向后兼容性。
- 将一些安全相关的算法细节从附录移动到标准，并将ClientKeyShare降级到附录。
- 添加带有Poly1305消息验证码的ChaCha20流加密。
- 添加Ed25519和Ed448数字签名算法。
- 添加x25519和x448密钥交换协议。
- 将支持加密服务器名称指示（**E**ncrypted**S**erver**N**ame**I**ndication, ESNI）。

### 密钥交换和密钥协商

在客户端和服务器开始交换TLS所保护的加密信息之前，他们必须安全地交换或协定加密密钥和加密数据时要使用的密码。用于密钥交换的方法包括：使用[RSA](https://baike.baidu.com/item/RSA)算法生成公钥和私钥（在TLS[握手](https://baike.baidu.com/item/握手)协议中被称为TLS_RSA），[Diffie-Hellman](https://baike.baidu.com/item/Diffie-Hellman)（在TLS握手协议中被称为TLS_DH），临时Diffie-Hellman（在TLS握手协议中被称为TLS_DHE），椭圆曲线迪菲-赫尔曼（在TLS握手协议中被称为TLS_ECDH），临时椭圆曲线Diffie-Hellman（在TLS握手协议中被称为TLS_ECDHE），匿名Diffie-Hellman（在TLS握手协议中被称为TLS_DH_anon）和预共享密钥（在TLS握手协议中被称为TLS_PSK）。

TLS_DH_anon和TLS_ECDH_anon的密钥协商协议不能验证服务器或用户，因为易受[中间人攻击](https://baike.baidu.com/item/中间人攻击)因此很少使用。只有TLS_DHE和TLS_ECDHE提供前向保密能力。

在交换过程中使用的[公钥](https://baike.baidu.com/item/公钥)/[私钥](https://baike.baidu.com/item/私钥)加密[密钥](https://baike.baidu.com/item/密钥)的长度和在交换协议过程中使用的公钥证书也各不相同，因而提供的**强健性**的安全。2013年7月，[Google](https://baike.baidu.com/item/Google)宣布向其用户提供的TLS加密将不再使用1024位公钥并切换到2048位，以提高安全性。 [1]

## TLS&SSL

1）SSL表示安全套接字层，而TLS表示传输层安全，前者由Netscape于1994年开发，旨在在客户端和服务器系统之间建立一种安全的通信方式。互联网工程任务组（IETF）对SSL协议进行了标准化，并且有两个版本，即SSL 1.0，该版本由于存在缺陷而没有公开发布，后来推出了SSL 2.0，但存在一些设计缺陷。结果，进行了升级和增强，以发布旨在修复早期缺陷的SSL 3.0。

2）另一方面，TLS于1999年发布，TLS 1.1等其他版本分别于2006年和2008年发布。但是，TLS 1.3于2018年8月发布，它具有增强的功能，其中删除了MD5和SHA-224，并在以前的配置中使用了数字签名。

3）SSL协议为密码套件提供支持，而TLS不提供任何此类支持，而是使用标准化协议，该协议使定义新的密码套件类别变得容易。

4）TLS协议不使用“无证书”警报消息，而是使用其他警报消息，与SSL协议不同，SSL协议仍然依赖“无证书”警报消息。
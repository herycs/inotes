# 加密

## 数字摘要

唯一对于消息或文本的固定长度值，由单向Hash函数对消息计算得出，也称为“数字指纹”

特点：

- 长度-固定
- 结果冲突-好的消息摘要算法很难发生碰撞
- 单向性-由内容产出数字指纹，而很难反向推出源文件（反推方式就是穷举所有可能的结果）

### MD5

message digest algorithm 5，消息摘要算法5

摘要长度：125

MD2，MD3，MD4改进而来，增加算法复杂度，不可逆性

```java
public static byte[] testMD5(String content) throws Exception {
    MessageDigest md5 = MessageDigest.getInstance("MD5");
    byte[] utf8s = md5.digest(content.getBytes("utf8"));
    return utf8s;
}
```

### SHA

secure hash algorithm，安全散列算法

SHA-1基于MD4，长度160

```java
public static byte[] testSHA1(String content) throws NoSuchAlgorithmException, UnsupportedEncodingException {
    MessageDigest md5 = MessageDigest.getInstance("SHA-1");
    byte[] utf8s = md5.digest(content.getBytes("utf8"));
    return utf8s;
}
```

### 16进制编码

java中1byte = 8 bit，且第一位为符号位，故转换时需要将第一位转换为对应的数值

```java
// byte -> hex
public static String bytesToHex(byte[] bytes) {
    StringBuilder builder = new StringBuilder();
    for (int i = 0; i < bytes.length; i++) {
        byte b = bytes[i];

        boolean negative = b < 0 ? true : false;
        int num = Math.abs(b);
        if (negative) num = num | 0x80;

        String temp = Integer.toHexString(num & 0xFF);
        if (temp.length() == 1) {
            builder.append("0");
        }
        builder.append(temp.toLowerCase());
    }
    return builder.toString();
}
// hex --> byte
public static String hexToBytes(String hex) {
    byte[] bytes = new byte[hex.length() / 2];
    for (int i = 0; i < hex.length(); i+=2) {
        String substring = hex.substring(i, i + 2);

        boolean negative = false;
        int inte = Integer.parseInt(substring, 16);
        if (inte > 127) negative = true;
        if (inte == 128) inte = -128;
        else if (negative) inte = 0 - (inte & 0x7F);

        byte b = (byte) inte;
        bytes[i / 2] = b;
    }
    return bytes.toString();
}
```

不排除byte的符号位则会出现问题，如下

```java
        byte n = (byte) 1010_1101;

        int num = n | 0x80;
        System.out.println(Integer.toHexString(num & 0xFF));
//        ed
        System.out.println(Integer.toHexString(n & 0xFF));
//        6d, 1010减一-》1001-》取反得到源码 0110--》6
```

### Base64

基于64个可打印字符表示2进制

每6个为一个单位---》对应一个可打印字符

具体可打印字符以具体系统而定

```java
public static String testBase64(byte[] bytes) {
    BASE64Encoder base64Encoder = new BASE64Encoder();
    return base64Encoder.encode(bytes);
}
public static byte[] testBase64Byte(String base64) throws IOException {
    BASE64Decoder base64Decoder = new BASE64Decoder();
    return base64Decoder.decodeBuffer(base64);
}
```

### 彩虹表破

### 解Hash算法

随着各种数据库沦陷彩虹表的累计越来越丰富，越来越准确，是一种不可忽视的威胁

## 对称加密算法

算法公开，计算量小，加密速度快，加密效率高

### DES算法

明文64位分组，秘钥64位（其中54位参与DES运算，第8、16、24、32、40、56、64位是校验位，使得每个秘钥有奇数个1），分组后的明文和56位的秘钥按位替代或交换的方法形成秘文

3DES，三条56位秘钥对数据进行三次加密

```java
// 生成DES秘钥
public static String getKeyDES() throws NoSuchAlgorithmException {
    KeyGenerator des = KeyGenerator.getInstance("DES");
    des.init(56);// 设置秘钥为54位
    SecretKey secretKey = des.generateKey();
    String s = testBase64(secretKey.getEncoded());
    return s;
}
// 秘钥转换为Base64格式存储的
public static SecretKey loadKeyDES(String basse64key) throws IOException {
    byte[] bytes = testBase64Byte(basse64key);
    SecretKey des = new SecretKeySpec(bytes, "DES");
    return des;
}
// DES加密
public static byte[] encryptDES(byte[] source, SecretKey key) throws Exception {
    Cipher des = Cipher.getInstance("DES");
    des.init(Cipher.ENCRYPT_MODE, key);
    byte[] bytes = des.doFinal(source);
    return bytes;
}
// DES解密
public static byte[] decryptDES(byte[] source, SecretKey key) throws Exception {
    Cipher des = Cipher.getInstance("DES");
    des.init(Cipher.DECRYPT_MODE, key);
    byte[] bytes = des.doFinal(source);
    return bytes;
}
```

#### 分组模式

#### EBC

优点：

1. 简单；
2. 有利于并行计算；
3. 误差不会被传送；

缺点：

1. 不能隐藏明文的模式；
2. 可能对明文进行主动攻击。

#### CBC

优点：

1. 不容易主动攻击,安全性好于ECB,适合传输长度长的报文,是**SSL**、**IPSec**的标准。

缺点：

1. 不利于并行计算；
2. 误差传递；
3. 需要初始化向量IV。

#### CFB

优点：

1. 隐藏了明文模式；
2. 分组密码转化为流模式；
3. 可以及时加密传送小于分组的数据。

缺点:

1. 不利于并行计算；
2. 误差传送：一个明文单元损坏影响多个单元；
3. 唯一的IV。

#### OFB

优点：

1. 隐藏了明文模式；
2. 分组密码转化为流模式；
3. 可以及时加密传送小于分组的数据。

缺点：

1. 不利于并行计算；
2. 对明文的主动攻击是可能的；
3. 误差传送：一个明文单元损坏影响多个单元。

#### CTR

加密方式：密码算法产生一个16 字节的伪随机码块流，伪随机码块与输入的明文进行异或运算后产生密文输出。密文与同样的伪随机码进行异或运算后可以重产生明文。

CTR 模式被广泛用于 ATM 网络安全和 IPSec应用中，相对于其它模式而言，CRT模式具有如下特点：

■硬件效率：允许同时处理多块明文 / 密文。

■ 软件效率：允许并行计算，可以很好地利用 CPU 流水等并行技术。

■ 预处理：算法和加密盒的输出不依靠明文和密文的输入，因此如果有足够的保证安全的存储器，加密算法将仅仅是一系列异或运算，这将极大地提高吞吐量。

■ 随机访问：第 i 块密文的解密不依赖于第 i-1 块密文，提供很高的随机访问能力

■ 可证明的安全性：能够证明 CTR 至少和其他模式一样安全（CBC, CFB, OFB, ...）

■ 简单性：与其它模式不同，CTR模式仅要求实现加密算法，但不要求实现解密算法。对于 AES 等加/解密本质上不同的算法来说，这种简化是巨大的。

■ 无填充，可以高效地作为流式加密使用。

### AES

advance encryption standard，高级加密标准

具有：强安全性，高性能，高效率，易用和灵活性等优点

比加密方式 128 192 256 三种，其他同DES

另外使用192,256位秘钥，由于美国加密软件出口控制，需要下载无政策和司法限制的文件，否则运行时会出现异常

```java
public static String genKeyAES() throws NoSuchAlgorithmException {
    KeyGenerator aes = KeyGenerator.getInstance("AES");
    aes.init(128); // 128 | 192 | 256
    SecretKey secretKey = aes.generateKey();
    String s = testBase64(secretKey.getEncoded());
    return s;
}
```

## 非对称加密

### RSA

名称来源三个开发者名称首字母

基于简单数论实时，两个大素数相乘容易，但反推很困难，乘积可公开作为加密密钥

使用公钥加密，私钥解密

### DSA

### ECDSA

## 数字签名

### MD5withRSA

采用MD5生成需要发送正文的数字摘要

使用RSA算法私钥对摘要算法加密

### SHA-1withRSA

SHA-1生成正文数字摘要，使用RSA对摘要加密

## 数字证书

### x.509证书标准

### 证书签发

### 证书校验

### 证书管理

自签名证书

OpenSSL

SSL协议库，密码算法库，与之相关的应用程序

## 摘要认证

为什么需要？

有的数据更加关注其正确性和准确性

原理

每次请求与响应依据一定规则生成数字摘要（涵盖客户端，服务端通信内容，盐）

实现

- 客户端参数摘要生成
- 服务端参数摘要生成
- 服务端响应摘要生成
- 客户端响应摘要校验

## 签名认证

原理

摘要认证在一定程度上可保证数据的安全性（盐不泄露）

实现

- 客户端参数签名生成
- 服务端参数签名校验
- 服务端参数签名生成
- 客户端参数签名校验

## HTTPS协议

### 原理

Hypertext Transfer Protocol over Secure Socket Layer

SSL / TLS 协议均可分为两层：

- Record Protocol，记录协议，完成数据加密解密，压缩，校验
- Handshake Protocol，握手协议，记录协议基础上，开始前进行加密算法协商，秘钥交换，通信双方认证

## OAuth

存在漏洞
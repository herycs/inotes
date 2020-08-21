# 攻击&防御

## XSS

Cross-Site-Scripting，跨站脚本攻击

<u>利用站点内信任用户</u>

原理：攻击者在网页中植入恶意代码，当用户打开网页，则开始在用户浏览器上执行，盗取用户cookie

示例：

```html
<html>
	<head>
	</head>
	<body>
		<!---
			预期此处显示name
			但若用户名是不合法字符，则此处会是诸如
			<script>while(true){alert("hello")}</script><!---
			的脚本
		--->
	</body>
</html>
```

### 防范

发生原因是用户输入的< > 代码作为真实代码执行了，将这样的字符进行转义即可

< ---> \&lt;

\> ---> \&gt;

\' ---> \&amp;

\" ---> \&quot;

## CRSF

cross site request forgery，跨站请求伪造

<u>冒充真实用户</u>

### 防御

- cookie设置为HttpOnly
- 增加token
- 通过referer记录请求源地址

## SQL注入攻击

### 注入攻击原理

### 防范

- 使用预编译语句
- ORM框架
- 避免密码明文存放
    - Hash彩虹表用于破解(逆向推理)
    - 加盐，提高其扰动性
- 异常处理，避免因为异常直接暴露给客户端导致反向推理出后台信息

## 文件上传漏洞

### 防范

- 文件类型白名单
- 文件大小限制
- 上传文件重命名（避免存储路径暴露）

文件类型检查，不仅仅依据于文件头的魔术用于检查文件类型

```java
package com.w.utils;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * @ClassName File
 * @Description [文件类型检查]
 * @Author ANGLE0
 * @Date 2020/8/21 16:28
 * @Version V1.0
 **/
public enum FileType {
    /*
        jpeg
     */
    JPEG("FFD8FF"),
    /*
        PNG
     */
    PNG("89504E47"),
    /*
        GIF
     */
    GIF("47494638"),
    /*
        TIFF
     */
    TIFF("49492A00"),
    /*
        BIT MAP
     */
    BMP("424D"),
    /*
        CAD
     */
    DWG("41433130"),
    /*
        photoshop
     */
    PSD("38425053"),
    /*
        XML
     */
    XML("3C3F786D6C"),
    /*
        HTML
     */
    HTML("68746D6C3E"),
    /*
        Adobe Acrobat
     */
    PDF("255044462D312E"),
    /*
        ZIP Archive
     */
    ZIP("504B0304"),
    /*
        RAR Archive
     */
    RAR("52617221"),
    /*
        WAVE
     */
    WAV("57415645"),
    /*
        AVI
     */
    AVI("41564920");

    private String value = "";

    private FileType(String fileType) {
        this.value  = fileType;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    private static String getFileHeader(String filePath) throws IOException {
        byte[] b = new byte[28];
        InputStream inputStream = new FileInputStream(filePath);
        inputStream.read(b, 0, 28);
        inputStream.close();
//        return bytes2hex(b);
        return null;
    }

    public static FileType getFileType(String filePath) throws IOException {
        String fileHeader = getFileHeader(filePath);
        if (fileHeader == null || fileHeader.length() == 0) {
            return null;
        }
        fileHeader = fileHeader.toUpperCase();
        FileType[] values = FileType.values();
        for (FileType type : values) {
            if (fileHeader.startsWith(type.getValue())) return type;
        }
        return null;
    }
}
```

图片文件压缩处理

imagemagick环境准备

jmagick提供imagemagick中缺失的头文件jni

## DDos

distributed Denial of Service，分布式拒绝服务攻击

### SYN Flood

伪造IP地址发起TCP连接，由于伪造的地址并不存在客户端故服务端的响应得不到确认，也就是握手只有两次

### DNS Query Flood

UDP Flood攻击方式的变形

向被攻击服务器发送海量域名解析请求（而这大量的请求所需要解析的域名随机生成，这会致使DNS服务器向其他服务器发起进一步解析请求，当达到一定数量，DNS会解析超时）

### CC攻击

challenge collapsar

基于应用层Http协议发起DDos攻击，成为Http flood

通过搜索网络上的大量的匿名HTTP代理，依据这些代理发起请求，防范方式很难在不影响业务情况下进行

## 其他

DNS域名劫持，CDN回源攻击，服务器权限升级，缓冲区溢出




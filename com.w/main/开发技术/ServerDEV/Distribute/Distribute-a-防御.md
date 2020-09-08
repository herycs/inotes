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

- 

## 文件上传漏洞





## 其他



### 浏览器脚本攻击

```shell


```




# URL

> URL：uniform resource locator（统一资源定位符）
>
> - http://www.baidu.com
>
> URI：Uniform Resource Identifier（统一资源标识符）
>
> - URI = URL+URN = http://www.baidu.com/ + test/index.html  = http://www.baidu.com/test/index.html

## 操作流程

无论采用哪种方式，处理流程都是不变的

1. 构建目标，首先需要一个地址（或者说目标），也就是构建URL对象对请求数据封装
2. 获取连接，基于URL构建连接
3. 进行通讯，获取流或者发送流
4. 结果解析，封装数据或者直接输出

## URL

### 作用

- openStream()方法与制定的URL建立连接并返回InputStream类的对象，支持数据的读取

### 内部属性

```java
static final String BUILTIN_HANDLERS_PREFIX = "sun.net.www.protocol";
static final long serialVersionUID = -7627629688361524110L;

private static final String protocolPathProp = "java.protocol.handler.pkgs";

private String protocol;

private String host;

private int port = -1;

private String file;

private transient String query;

private String authority;

private transient String path;

private transient String userInfo;

private String ref;

transient InetAddress hostAddress;

transient URLStreamHandler handler;

private int hashCode = -1;

private transient UrlDeserializedState tempState;
```

### 构建

| **构造方法**                                                 | **说明**                                                |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| public URL (String spec)                                     | 通过一个表示 URL 地址的字符串可以构造一个 URL 对象      |
| public URL(URL context,String spec)                          | 使用基本地址和相对 URL 构造一个 URL 对象                |
| public URL(String protocol,String host,String file)          | 使用指定的协议、主机名和文件名创建一个 URL 对象         |
| public URL(String protocol,String host,int port,String file) | 使用指定的协议、主机名、端口号和文件名创建一个 URL 对象 |

### 常用方法

| public String getProtocol()  | 获取该 URL 的协议名                            |
| ---------------------------- | ---------------------------------------------- |
| public String getHost()      | 获取该 URL 的主机名                            |
| public int getPort()         | 获取该 URL 的端口号，如果没有设置端口，返回 -1 |
| public String getFile()      | 获取该 URL 的文件名                            |
| public String getRef()       | 获取该 URL 在文件中的相对位置                  |
| public String getQuery()     | 获取该 URL 的查询信息                          |
| public String getPath()      | 获取该 URL 的路径                              |
| public String getAuthority() | 获取该 URL 的权限信息                          |
| public String getUserInfo()  | 获得使用者的信息                               |
| public String getRef()       | 获得该 URL 的锚点                              |

### 尝试

```java
main(){
    try{
        URL url  = new URL("http://www.baidu.com");
        InputStream in = url.openStream();

        InputStreamReader reader = new InputStreamReader(in);
        BufferedReader readbuf = new BufferedReader(reader);

        String st = null;
        while ((st = readbuf.readLine()) != null ){
            System.out.println(builder);
        }
    }//...省略其他catch代码
}
```



## URLConnection

> 支持通信交互

### 作用

- openConnection()方法会创建一个URLConnection类的对象，此对象在本地机和URL指定的远程节点建立一条HTTP协议的数据通道，可进行双向数据传输。

### 常用方法

| 方法                                             | 作用                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| void addRequestProperty(String key,String value) | 添加由键值对指定的一般请求属性。key 指的是用于识别请求的关键字 （例如 accept），value 指的是与该键关联的值 |
| void connect()                                   | 打开到此 URL 所引用的资源的通信链接（如果尚未建立这样的链接） |
| Object getConnection()                           | 检索此 URL 链接的内容                                        |
| InputStream getInputStream()                     | 返回从此打开的链接读取的输入流                               |
| OutputStream getOutputStream()                   | 返回写入到此链接的输出流                                     |
| URL getURL()                                     | 返回此 URLConnection 的 URL 字段的值                         |

### 示例

#### get

```java
main(){
    try {
        URL url = new URL("http://www.baidu.com");//构建URL对象
        URLConnection urlCon = url.openConnection();//建立链接
        HttpURLConnection httpCon = (HttpURLConnection) urlCon;//分装为Http协议下的链接

        int responCode = httpCon.getResponseCode();//查看响应码
        if (responCode == HttpURLConnection.HTTP_OK){//链接成功

            InputStream in = httpCon.getInputStream();//获取输入流
            InputStreamReader reader = new InputStreamReader(in);//获取读入流
            BufferedReader readbuf = new BufferedReader(reader);

            String str = null;
            while ((str = readbuf.readLine()) != null){
                System.out.println(str);//读入的数据打印
            }
        }
    }//...省略其他catch代码
}
```

post

```java
main(){
    try{
        //创建ULR con对象
        URL url = new URL(urlString);
        //打开URL连接
        HttpURLConnection con = (HttpURLConnection) url.openConnection();
        //设置请求头等信息
        con.setDoOutput(true);
        con.setDoInput(true);
        con.setRequestMethod("POST");
        con.setRequestProperty("connection", "Keep-Alive");
        con.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
        con.setConnectTimeout(4000);//连接超时时间
        //建立连接
        con.connect();
        //发送请求参数
        //...获取输出流，缓存请求参数，请求参数准备完毕，输出数据，关闭输出流
        // 读取响应
        if (con.getResponseCode() != 200)
            System.out.println("连接失败");
        else{
            readBuf = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String str = null;
            while ((str = readBuf.readLine()) != null){
                System.out.println(str.toString());
                resultString.append(str);
            }
        }
        //关闭流
    }
}
```


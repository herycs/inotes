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

# 协议&标准化

## 协议标准化

- 国际标准化组织--ISO
- 国际电信联盟-电信标准部--ITU-T(CCITT)
- 因特网协会--ISOC（IETF）
- 电气和电子工程师协会--IEEE
- 电子工业协会--EIA/TIA
- 因特网名字与号码指派公司--ICANN

### 因特网标准化工作流程

1. 因特网草案，Internet Draft
2. 建议标准，Proposed Standard,RFC文档
3. 草案标准，Draft Standard
4. 因特网标准，Internet Standard

> 另外三种RFC，历史的（historic)，实验的(experimental)，提供信息的(informational)

### 标准化组织

国际标准化组织ISO：OSI参考模型，HDLC协议

国际电信联盟ITU：指定通信规则

国际电信工程师协会IEEE：学术机构，IEEE802系列标准，5G

## 网络协议

### 定义

- 为进行网络中的数据交换而建立的规则，标准或约定

### 三要素

- 语法：数据与控制信息的结构与格式
- 语义：发出何种控制信息，完成何种动作以及做出何种响应
- 定时：事件实现顺序的详细说明

## 协议分层

### 好处

- 简化系统设计，模块化易于维护，系统的更新成本和效率

### 负面影响

- 信息冗余，性能降低
- 层次难以确定
- 协议首部越来越大

### 设计原则

- 功能模块化：取出不同网络应用之间以及网络应用与网络技术之间的紧密耦合
- 端到端原则：划分了网络功能关键模块应当在哪些模块中实现
- 定义好接口将复杂系统分隔开

## 几种体系结构

### OSI七层

#### 目的

为了支持互连互通

#### 缺点

1. 理论完善，但实践并不好
2. 实现复杂，效率低
3. 指定周期长，世面已经有了成熟产品
4. 分层划分不是很清晰，有冗余

#### 分层

#### 逐层封装的原因

> 水平协议，垂直服务

- 对等层需要交换本层控制信息
- 每一层对等层之间有规范PDU格式的协议
- 每一层有对应的控制信息

#### 各层概述

| 层         | 数据单元             | 作用                                | 协议                                      | 设备                 |
| ---------- | -------------------- | ----------------------------------- | ----------------------------------------- | -------------------- |
| 应用层     | 应用协议数据单元APDU | 允许访问OSI环境的手段               | HTTP，FTP、DNS、SMTP、WWW、Telnet、NFS    |                      |
| 表示层     | 表示协议数据单元PPDU | 对数据进行翻译、加密和压缩          | ASII、JPEG、MPEG                          |                      |
| 会话层     | 会话协议数据单元SPDU | 建立、管理和终止会话                | SQL、RPC、NETBIOS、NFS、                  |                      |
| 传输层     | 段Segment            | 提供端到端的可靠报文传递和错误恢复  | TCP、UDP、SPX                             |                      |
| 网络层     | 包Packet             | 负责数据包从源到宿的传递和网际互连  | IP、ICMP、ARP、RARP、RIP、OSPF、IPX、IGRP | 路由器               |
| 数据链路层 | 帧Frame              | 将比特组装成帧和点到点的传递        | PPP、VLAN、MACFR、HDLC                    | 网桥，交换机         |
| 物理层     | 比特Bit              | 通过媒介传输比特,确定机械及电气规范 | RJ45、CLOCK、IEEE802.3                    | 中继器，集线器，网关 |

#### 传输过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727222933619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)

### TCP/IP分层参考

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730152015855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70)
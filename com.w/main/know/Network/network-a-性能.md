# 分组交换网的性能指标

## 带宽和吞吐量

- 速率：信道上的传输数据的速率，称数据率，比特率
- 网络带宽：链路在一段特定时间内所能传送的比特数的额定值，bps
- 吞吐量：网络在单位时间内无差别的传输数据的能力
- 瓶颈链路：路径中可用带宽最小的链路
- 可用带宽：带宽与干扰流量之差
- 比特时间：传送一个比特的所花费的时间，对信道而言比特时间=1/带宽
- 带宽：高低频之差，频带宽度（Hz）

## 常用带宽单位

- 千比每秒，kb/s(10^3 b/s)
- 兆比每秒，Mb/s(10^6 b/s)
- 吉比每秒，Gb/s(10^9 b/s)
- 太比每秒，Tb/s(10^12 b/s)

> K = 2^10 = 1024 bit
>
> M = 2^20 bit
>
> G = 2^30 bit
>
> T = 2^40 bit

## 四种延时

```shell
场景：A ---> B
```

在A点的*处理时延*和*排队时延*，在A节点到链路的发送器中产生的*传输时延*（发送时延），在链路上产生的（*传播时延*），单位：s

- 处理时延：bit差错检测以及链路选择
- 排队时延：等待输出链路传输时间（取决于路由器拥塞等级）
- 传输时延：分组长度（bit） / 链路带宽（bps）
- 传播时延：物理链路长度 / 传播速度

## 跳与路径

## 时延带宽积

> 应用进程之间的信道看做中空的管道，时延相当于管道长度，带宽相当于管道直径
>
> **时延带宽积**相当于管道**容积**：管道可容纳的比特数

- 信道的比特长度 = 传播延时x信道宽度
- 数据吞吐率或吞吐量：单位时间内通过网路的数据量
- 信道利用率：某信道有百分之几的时间是利用的，=实际通过的数据量/理论上应该通过的数据量
- 网络利用率是全网络的信道利用率的加权平均值

## 往返时延RTT

发送方发送到接收到反馈的时间

T = 发送方接收到接收方反馈时间T2 - 发送T1

## 利用率

信道利用率：有数据传输时间 / 总时间

网络利用率：信道利用率加权平均值
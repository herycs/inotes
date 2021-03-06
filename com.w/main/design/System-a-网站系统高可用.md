# 网站系统高可用

> 事物总是先求生存，然后求发展

## 评价

### 度量

| 可用性 | 不可用时间(年度不可用时间/小时) | 语义           |
| ------ | ------------------------------- | -------------- |
| 2个9   | <88                             | 基本可用       |
| 3个9   | <9                              | 较高可用       |
| 4个9   | <53                             | 自动回复高可用 |
| 5个9   | <5(分钟)                        | 极高可用       |

### 考核

故障权重表

| 分类       | 描述                                     | 权重 |
| ---------- | ---------------------------------------- | ---- |
| 事故级故障 | 严重故障，整体网站不可用                 | 100  |
| A类故障    | 网站访问不顺畅或者核心功能不可用         | 20   |
| B类故障    | 非核心功能不可用，核心功能少数用户不可用 | 5    |
| C类故障    | 以上故障之外其他故障                     | 1    |

故障分计算：
$$
故障分 = 故障时间（分钟） * 故障权重
$$

## 实现

### 负载均衡

通过负载均衡实现无状态的故障转移

### 集群Session 管理

1. Session复制
2. Session绑定
3. Cookie记录Session
4. Session服务器

### 高可用的服务

1. 分级管理：核心使用优等资源，依次分配
2. 超时设置
3. 异步调用
4. 服务降级
5. 幂等性设计

### 高可用数据

#### CAP原理

一致（Consistency） | 可用（Availibity） | 分区耐受（Patition Tolerance，跨域网络分区伸缩性）

数据一致性：

- 数据强一致性
- 数据用户一致
- 数据最终一致

### 数据备份

热备：

- 异步热备
- 同步热备

### 失效转移

1. 失效转移
2. 访问转移
3. 数据恢复

## 高可用网站质量保证

### 网站发布

#### 自动化测试

#### 预发布验证

#### 代码控制

1. 主干发布，分支发布
2. 分支发布，主干发布

#### 自动发布

#### 灰度发布

每天发布一部分，出现问题则回滚

### 完整监控

#### 数据采集

1. 用户行为日志
    - 服务器端搜集
    - 客户端信息搜集
2. 服务性能监控
3. 运行报告

#### 监控管理

- 系统报警
- 失效转移
- 自动优雅降级
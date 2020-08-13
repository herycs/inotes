# Lua

## 认知

为了执行Lua，Redis在服务器中内嵌了Lua环境

修改Lua环境整个过程：

1. 创建基础Lua环境
2. 载入多个函数库到Lua环境中
3. 创建全局表格Redis
4. 使用Redis自带随机函数替换Lua原有随机函数
5. 创建辅助排序函数，对命令排序，消除命令不确定性
6. 创建错误报告辅助函数
7. 对Lua全局环境进行保护，防止用户在执行Lua脚本过程中将额外的全局变量添加到环境中
8. 将修改完成的Lua环境保存到服务器的Lua属性中，等待执行Lua脚本

## 过程

### 创建Lua环境

调用lua基础函数，lua_open

### 载入函数库

基础库：核心函数

表格库：用于表格处理的基础函数

字符串库：

数学库：

调试库：

Lua CJSON库：

Struct库：

Lua cmsgpack库：

### 创建Redis全局表格

表格包含以下函数

- 执行Redis命令
- 记录Redis日志
- 计算SHAl校验和
- 返回错误信息

### 确保数据一致性

#### 替换Lua自带随机函数

为确保Lua在不同机器上执行出相同结果，Redis要求Lua脚本内函数必须是无副作用的纯函数

替换之后，math.random总产生相同的随机数序列，使用math.randomseed显式的修改seed

#### 创建排序函数

集合顺序不唯一

#### 创建错误报告函数

#### 保护Lua全局环境

当创建全局变量时会发生错误，但允许修改全局变量（这点操作时需要谨慎些）

## Lua环境协作

伪客户端 + 存储Lua脚本的字典

### 伪客户端

脚本调用者-》Lua环境-》伪客户端-》命令执行器-》执行结果

脚本调用者《-Lua环境《-伪客户端《-命令执行器《-执行结果

### lua_script字典

k-v，k - lua脚本校验和，SHA1校验和

所有被EVAL执行过的Lua脚本，所有被SCRIPT LOAD载入的Lua脚本保存到Lua_scripts字典中

作用：

- 实现scripts exists
- 实现脚本复制

## EVAL命令

### 执行过程

1. 依据Lua脚本，定义Lua函数
2. 脚本保存到字典中
3. 执行Lua环境中定义的函数，执行客户端给定的脚本

## 脚本管理

### Script Flush

### Script Exists

### Script Load

### Script kill

### 脚本复制

EVAL

Script Flush

Script Load
# MySQL读写分离

## 基础概念

### 出现的原因

- 并发环境下保证数据的安全性以及系统的性能，若出现操作表导致无法读取，这是很影响业务的
- 做数据的热备
- 架构的扩展。业务量过大，单机的IO性能无法支持大量IO，通过多库的存储，降低磁盘I/O访问的频率，提高单个机器的I/O性能。

### 策略

- master数据库处理写操作，slave数据库处理读操作。
- master将写操作的变更同步到各个slave节点。减少了从节点的业务量。
    - 数据对象是：主数据库中的**所有数据库**或者**特定的数据库**，或者**特定的表**

## 读写分离能提高系统性能的原因

​		1、物理服务器增加，机器处理能力提升。拿硬件换性能。

​		2、主从只负责各自的读和写，极大程度缓解X锁和S锁争用。

​		3、slave可以配置myiasm引擎，提升查询性能以及节约系统开销。

​		4、master直接写是并发的，slave通过主库发送来的binlog恢复数据是异步。

​		5、slave可以单独设置一些参数来提升其读的性能。

​		6、增加冗余，提高可用性。

## 具体实现

- master服务器将数据的改变记录二进制binlog日志
- slave服务器定时对master二进制日志进行探测，如果发生改变，则开始一个I/OThread请求master二进制日志
- 同时主节点为每个I/O线程启动一个dump线程，用于向其发送二进制事件，并保存至从节点本地的中继日志relay log中，从节点将启动SQL线程从中继日志中读取二进制日志，更新数据，最后I/OThread和SQLThread将进入睡眠状态，等待下一次被唤醒。

> slave开启两个线程：IO线程和SQL线程。
>
> - 其中：IO线程负责读取master的binlog内容到中继日志relay log里；
> - SQL线程负责从relay log日志里读出binlog内容，并更新到slave的数据库里，这样就能保证slave数据和master数据保持一致了。
>
> 版本问题：master V <= slave V
> master和slave两节点间时间需同步

### 操作步骤

1、从库通过手工执行change  master to 语句连接主库，提供了连接的用户一切条件（user 、password、port、ip），并且让从库知道，二进制日志的起点位置（file名 position 号）；    start  slave

2、从库的IO线程和主库的dump线程建立连接。

3、从库根据change  master  to 语句提供的file名和position号，IO线程向主库发起binlog的请求。

4、主库dump线程根据从库的请求，将本地binlog以events的方式发给从库IO线程。

5、从库IO线程接收binlog  events，并存放到本地relay-log中，传送过来的信息，会记录到master.info中

6、从库SQL线程应用relay-log，并且把应用过的记录到relay-log.info中，默认情况下，已经应用过的relay 会自动被清理purge

### mysql主从形式

一主一从

主主复制

一主多从

多主一从

联级复制

## 操作

### 在master和slave上配置主从复制

#### master

配置

```shell
#修改配置文件，执行以下命令打开mysql配置文件
vi /etc/my.cnf
#在mysqld模块中添加如下配置信息
log-bin=master-bin #二进制文件名称
binlog-format=ROW  #二进制日志格式，有row、statement、mixed三种格式，row指的是把改变的内容复制过去，而不是把命令在从服务器上执行一遍，statement指的是在主服务器上执行的SQL语句，在从服务器上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。mixed指的是默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。
server-id=1		   #要求各个服务器的id必须不一样
binlog-do-db=school   #同步的数据库名称
```

授权子数据库

```sql
--授权操作
set global validate_password_policy=0;
set global validate_password_length=1;
grant replication slave on *.* to 'root'@'%' identified by '123';
--刷新权限
flush privileges;
```

验证

```shell
#重启mysql服务
service mysqld restart
#登录mysql数据库
mysql -uroot -p
#查看master的状态
show master status；
```

### slave

配置

```shell
#修改配置文件，执行以下命令打开mysql配置文件
vi /etc/my.cnf
#在mysqld模块中添加如下配置信息
log-bin=master-bin	#二进制文件的名称
binlog-format=ROW	#二进制文件的格式
server-id=2			#服务器的id
```

验证

```shell
#重启mysql服务
service mysqld restart
#登录mysql
mysql -uroot -p
#连接主服务器
change master to master_host='192.168.150.11',master_user='root',master_password='123',master_port=3306,master_log_file='master-bin.000001',master_log_pos=334;
#启动slave
start slave
#查看slave的状态
show slave status\G(注意没有分号)
```

### 进行proxy的相关配置

#### 安装及配置

> [mysql-proxy下载地址](https://downloads.mysql.com/archives/proxy/#downloads)

```sql
--进入mysql-proxy的目录
cd mysql-proxy
--创建目录
mkdir conf
mkdir logs
--添加环境变量
#打开/etc/profile文件
vi /etc/profile
#在文件的最后添加命令
export PATH=$PATH:/root/mysql-proxy/bin
--更新
source /etc/profile
--进入conf目录，创建文件并添加一下内容
    vi mysql-proxy.conf
   --添加内容
    [mysql-proxy]
    user=root
    proxy-address=192.168.85.14:4040
    proxy-backend-addresses=192.168.85.11:3306
    proxy-read-only-backend-addresses=192.168.85.12:3306
    proxy-lua-script=/root/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua
    log-file=/root/mysql-proxy/logs/mysql-proxy.log
    log-level=debug
    daemon=true
--开启mysql-proxy
    mysql-proxy --defaults-file=/root/mysql-proxy/conf/mysql-proxy.conf
```

#### 验证安装

```shell
#查看是否安装成功，打开日志文件
cd /root/mysql-proxy/logs
tail -100 mysql-proxy.log
```

### 进行连接

```sql
--连接代理
mysql -uroot -p -h 192.168.43.122 -P 4040
```

## Amoeba配置读写分离

​		Amoeba(变形虫)项目，专注 分布式数据库 proxy 开发。座落与Client、DB Server(s)之间。对客户端透明。具有负载均衡、高可用性、sql过滤、读写分离、可路由相关的query到目标数据库、可并发请求多台数据库合并结果。

主要解决：

降低 数据切分带来的复杂多数据库结构

 提供切分规则并降低 数据切分规则 给应用带来的影响

 降低db 与客户端的连接数

 读写分离

### 为什么要用Amoeba

目前要实现mysql的主从读写分离，主要有以下几种方案：

- 通过程序实现，网上很多现成的代码，比较复杂，如果添加从服务器要更改多台服务器的代码。

- 通过mysql-proxy来实现，由于mysql-proxy的主从读写分离是通过lua脚本来实现，目前lua的脚本的开发跟不上节奏，而写没有完美的现成的脚本，因此导致用于生产环境的话风险比较大，据网上很多人说mysql-proxy的性能不高。

- 自己开发接口实现，这种方案门槛高，开发成本高，不是一般的小公司能承担得起。

- 利用阿里巴巴的开源项目Amoeba来实现，具有负载均衡、高可用性、sql过滤、读写分离、可路由相关的query到目标数据库，并且安装配置非常简单。国产的开源软件，应该支持，目前正在使用，不发表太多结论，一切等测试完再发表结论吧，哈哈！

### amoeba安装

- 安装
- 配置amoeba的配置文件

**dbServers.xml**

```xml
<?xml version="1.0" encoding="gbk"?>
<!DOCTYPE amoeba:dbServers SYSTEM "dbserver.dtd">
<amoeba:dbServers xmlns:amoeba="http://amoeba.meidusa.com/">
		
	<dbServer name="abstractServer" abstractive="true">
		<factoryConfig class="com.meidusa.amoeba.mysql.net.MysqlServerConnectionFactory">
			<property name="connectionManager">${defaultManager}</property>
			<property name="sendBufferSize">64</property>
            <property name="receiveBufferSize">128</property>
            
            <property name="port">3306</property>
			<property name="schema">school</property>	
            <property name="user">root</property>	
			<property name="password">123</property>
            
		</factoryConfig>

		<poolConfig class="com.meidusa.toolkit.common.poolable.PoolableObjectPool">
			<property name="maxActive">500</property>
			<property name="maxIdle">500</property>
			<property name="minIdle">1</property>
			<property name="minEvictableIdleTimeMillis">600000</property>
			<property name="timeBetweenEvictionRunsMillis">600000</property>
			<property name="testOnBorrow">true</property>
			<property name="testOnReturn">true</property>
			<property name="testWhileIdle">true</property>
		</poolConfig>
	</dbServer>

	<dbServer name="writedb"  parent="abstractServer">
		<factoryConfig>
			<!-- mysql ip -->
			<property name="ipAddress">192.168.85.11</property>
		</factoryConfig>
	</dbServer>
	
	<dbServer name="slave"  parent="abstractServer">
		<factoryConfig>
			<!-- mysql ip -->
			<property name="ipAddress">192.168.85.12</property>
		</factoryConfig>
	</dbServer>
	<dbServer name="myslave" virtual="true">
		<poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
			<!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
			<property name="loadbalance">1</property>
			<!-- Separated by commas,such as: server1,server2,server1 -->
			<property name="poolNames">slave</property>
		</poolConfig>
	</dbServer>
</amoeba:dbServers>
```

**amoeba.xml**

```xml
<?xml version="1.0" encoding="gbk"?>
<!DOCTYPE amoeba:configuration SYSTEM "amoeba.dtd">
<amoeba:configuration xmlns:amoeba="http://amoeba.meidusa.com/">
	<proxy>
		<!-- service class must implements com.meidusa.amoeba.service.Service -->
		<service name="Amoeba for Mysql" class="com.meidusa.amoeba.mysql.server.MySQLService">
			<!-- port -->
			<property name="port">8066</property>
			
			<!-- bind ipAddress -->
			<!-- 
			<property name="ipAddress">127.0.0.1</property>
			 -->
			<property name="connectionFactory">
				<bean class="com.meidusa.amoeba.mysql.net.MysqlClientConnectionFactory">
					<property name="sendBufferSize">128</property>
					<property name="receiveBufferSize">64</property>
				</bean>
			</property>
			
			<property name="authenticateProvider">
				<bean class="com.meidusa.amoeba.mysql.server.MysqlClientAuthenticator">
					<property name="user">root</property>
					<property name="password">123</property>
					<property name="filter">
						<bean class="com.meidusa.toolkit.net.authenticate.server.IPAccessController">
							<property name="ipFile">${amoeba.home}/conf/access_list.conf</property>
						</bean>
					</property>
				</bean>
			</property>
		</service>
		
		<runtime class="com.meidusa.amoeba.mysql.context.MysqlRuntimeContext">
			<!-- proxy server client process thread size -->
			<property name="executeThreadSize">128</property>
			<!-- per connection cache prepared statement size  -->
			<property name="statementCacheSize">500</property>
			<!-- default charset -->
			<property name="serverCharset">utf8</property>
			<!-- query timeout( default: 60 second , TimeUnit:second) -->
			<property name="queryTimeout">60</property>
		</runtime>
		
	</proxy>
	
	<!-- 
		Each ConnectionManager will start as thread
		manager responsible for the Connection IO read , Death Detection
	-->
	<connectionManagerList>
		<connectionManager name="defaultManager" class="com.meidusa.toolkit.net.MultiConnectionManagerWrapper">
			<property name="subManagerClassName">com.meidusa.toolkit.net.AuthingableConnectionManager</property>
		</connectionManager>
	</connectionManagerList>
	
		<!-- default using file loader -->
	<dbServerLoader class="com.meidusa.amoeba.context.DBServerConfigFileLoader">
		<property name="configFile">${amoeba.home}/conf/dbServers.xml</property>
	</dbServerLoader>
	
	<queryRouter class="com.meidusa.amoeba.mysql.parser.MysqlQueryRouter">
		<property name="ruleLoader">
			<bean class="com.meidusa.amoeba.route.TableRuleFileLoader">
				<property name="ruleFile">${amoeba.home}/conf/rule.xml</property>
				<property name="functionFile">${amoeba.home}/conf/ruleFunctionMap.xml</property>
			</bean>
		</property>
		<property name="sqlFunctionFile">${amoeba.home}/conf/functionMap.xml</property>
		<property name="LRUMapSize">1500</property>
		<property name="defaultPool">writedb</property>
		
		<property name="writePool">writedb</property>
		<property name="readPool">myslave</property>
		<property name="needParse">true</property>
	</queryRouter>
</amoeba:configuration>
```

### 启动amoeba

```shell
/root/amoeba-mysql-3.0.5-RC/bin/launcher
```

### 验证

```sql
分别在主从数据库操作
--将master服务停止，数据修改失败，但是能够查询
--将slave服务停止，数据修改成功，但是不能够查询
```


# firewall

> iptables

## 查看

### 服务

systemctl status firewalld.service

### firewall状态

firewall-cmd --state

### 防火墙管理

firewall-cmd --stop

firewall-cmd --start

firewall-cmd --restart

firewall-cmd --reload

### 查看规则

firewall-cmd --list-all

firewall-cmd --list-ports

端口是否开放

firewall-cmd --query-port=8080/tcp

### 端口管理

firewall-cmd --zone=public --add-port=80/tcp --permanent

firewall-cmd --permanent --add-port=8080/tcp

firewall-cmd --permanent --remove-port=8080/tcp
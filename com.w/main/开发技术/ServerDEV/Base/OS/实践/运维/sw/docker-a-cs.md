# docker-a-cs

## 创建CA秘钥文件夹

```shell
mkdir /usr/local/ca
cd /usr/local/ca/
```

## 创建密码

```shell
openssl genrsa -aes256 -out ca-key.pem 4096
//提供密码、国家、省、市、组织信息
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
[密码]
CN
sx
hz
social
FF
key
1@qq.com
openssl genrsa -out server-key.pem 4096
//$HOST为服务器外网或域名
openssl req -subj "/CN=122.51.8.208" -sha256 -new -key server-key.pem -out server.csr
```

## 创建白名单将Docker守护程序密钥的扩展使用属性设置为仅用于服务器身份验

```shell
//host,一下命令格式为：IP：HOST,IP:x.x.x.x
echo subjectAltName = IP:122.51.8.208,IP:0.0.0.0>>extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
```

## 签名证书

```shell
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \-CAcreateserial -out server-cert.pem -extfile extfile.cnf
//为客户端生成key
openssl genrsa -out key.pem 4096
//执行
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth >> extfile.cnf
//键入密码
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \-CAcreateserial -out cert.pem -extfile extfile.cnf
```

## 清理现场并增加安全性

```shell
rm -v client.csr server.csr
//修改文件权限
chmod -v 0400 ca-key.pem key.pem server-key.pem
//证书权限
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

## 证书移交至docker

```shell
cp server-*.pem  /etc/docker/
//配置
vim /lib/systemd/system/docker.service
//修改ExecStart
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/etc/docker/ca.pem --tlscert=/etc/docker/server-cert.pem --tlskey=/etc/docker/server-key.pem -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock
//重载
systemctl daemon-reload
systemctl restart docker
```

## 下载证书到本地

```shell
ca.pem
cert.pem
key.pem
```

## Idea添加docker认证

setting->B,E,D->Docker->{url:https://IP:2376,Cert:本地秘钥}
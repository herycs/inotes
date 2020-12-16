# 查询IP

hostname -I

ifconfig

您可以使用以下任何命令获取您的 IP ：

```shell
dig ANY +short @ resolver2.opendns.com myip.opendns.com 

dig ANY +short @ resolver2.opendns.com myip.opendns.com 

dig ANY +short @ ns1-1.akamaitech.net ANY whoami.akamai.net 
```

有许多在线 HTTP/HTTPS 服务可以使用您的公共 IP 地址进行响应。这里是其中的一些：

```shell
curl -s http://tnx.nl/ip

curl -s https://checkip.amazonaws.com

curl -s api.infoip.io/ip

curl -s ip.appspot.com

wget -O - -q https://icanhazip.com/
```

你甚至可以在你 ~/.bashrc 或 ~/.zshrc 文件创建一个别名，以后您不必键入并记住一个很长的命令。例如，您可以添加以下别名：

```shell
alias pubip='dig ANY +short @resolver2.opendns.com myip.opendns.com'
```
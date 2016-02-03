---
layout: post
title: VPS OpenVPN安装配置教程(基于Debian/Ubuntu)
category: 技术
tags: Concept
description: VPS OpenVPN安装配置教程(基于Debian/Ubuntu)
---

##1、安装

```
apt-get install openvpn udev lzop
```
##2、使用easy-rsa生成服务端证书

###将OpenVPN所需的配置文件复制到/etc/openvpn/下面：

```
cp -r /usr/share/doc/openvpn/examples/easy-rsa/ /etc/openvpn/
```
###生产CA证书：

```
cd /etc/openvpn/easy-rsa/2.0
source vars
./clean-all
./build-ca
```
./build-ca时会提示输入一些信息，可以都直接回车按默认信息。

###生成服务器端证书和密钥，server为名字可以自定义：

```
./build-key-server server
```
此步也是会提示输入一些信息，前面的信息直接回车按默认信息，提示Sign the certificate? [y/n]:时输入y，提示1 out of 1 certificate requests certified, commit? [y/n] 也是输入y。

###生成客户端证书和密钥，client为名字可以自定义，注意前面的./build-key-server与./build-key client输入的名字不能相同：

```
./build-key client
```
前面的信息直接回车按默认信息，提示Sign the certificate? [y/n]:时输入y，提示1 out of 1 certificate requests certified, commit? [y/n] 也是输入y

生成其他的客户端就是执行：./build-key 你想添加的客户端的名字。

生成的证书和密钥存放在/etc/openvpn/easy-rsa/2.0/keys/下面。

###生成Diffie Hellman参数：

```
./build-dh
```
##3、配置OpenVPN服务

###编辑/etc/openvpn/server.conf 文件，如果没有可以创建一个，加入下面的内容：

```
local 服务器IP
port 8080 #端口，需要与客户端配置保持一致
proto udp #使用协议，需要与客户端配置保持一致
dev tun #也可以选择tap模式

ca /etc/openvpn/easy-rsa/2.0/keys/ca.crt
cert /etc/openvpn/easy-rsa/2.0/keys/server.crt
key /etc/openvpn/easy-rsa/2.0/keys/server.key
dh /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem
ifconfig-pool-persist ipp.txt

server 10.168.1.0 255.255.255.0 #给客户的分配的IP段，注意不要与客户端网段冲突！

push "redirect-gateway"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"

client-to-client
;duplicate-cn
keepalive 20 60

comp-lzo
max-clients 50

persist-key
persist-tun

status openvpn-status.log
log-append openvpn.log

verb 3
mute 20
```

按上述说明修改服务器IP，复制到VPS上是可以把注释信息删除。

###安装iptables

```
apt-get install iptables   #如果已经安装可以跳过
```
###设置IP转发

```
iptables -t nat -A POSTROUTING -s 10.168.0.0/16 -o eth0 -j MASQUERADE
iptables-save > /etc/iptables.rules
```
上面的eth0要替换为你的网卡标识，可以通过ifconfig查看。

在/etc/network/if-up.d/目录下创建iptables文件，内容如下：

```
#!/bin/sh
iptables-restore < /etc/iptables.rules
```
给脚本添加执行权限：
```

chmod +x /etc/network/if-up.d/iptables
```
修改/etc/sysctl.conf的内容为：

```
net.ipv4.ip_forward = 1
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
```
重新载入/etc/sysctl.conf使其生效，执行如下命令：

```
sysctl -p
```
###重启OpenVPN及网络：

```
/etc/init.d/openvpn restart
/etc/init.d/networking restart
```
##4、安装配置OpenVPN客户端

###下载客户端

打开http://openvpn.net/download.html，点击Windows Installer后的链接，下载OpenVPN Windows客户端。

下载完成后，安装，安装中的选项全部按默认即可。

###下载客户端证书及密钥：

证书和密钥存放在/etc/openvpn/easy-rsa/2.0/keys/下面，可以使用winscp链接到VPS上下载。

将/etc/openvpn/easy-rsa/2.0/keys/下面的ca.crt、client.crt、client.key下载到C:\Program Files\OpenVPN\config 下面。

###创建客户端配置文件

在C:\Program Files\OpenVPN\config 下面创建一个linode.ovpn的文件，添加如下内容：

```
client
dev tun       #要与前面server.conf中的配置一致。
proto udp              #要与前面server.conf中的配置一致。
remote 服务器IP 8080    #将服务器IP替换为你的服务器IP，端口与前面的server.conf中配置一致。
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
ns-cert-type server
redirect-gateway
keepalive 20 60
#tls-auth ta.key 1
comp-lzo
verb 3
mute 20
route-method exe
route-delay 2
```
##5、OpenVPN客户端连接测试：

运行OpenVPN GUI，会在屏幕右下角的系统托盘区，右击该图标，会在菜单中出现我们添加的服务器，点击Connect，OpenVPN客户端就会开通链接OpenVPN服务器，过一会儿，OpenVPN图标变成绿色就是链接成功了。

如果想实现国内网站不走VPN，国外网站走VPN可以看一下：http://code.google.com/p/chnroutes/wiki/Usage 这个教程。
---
title: VPN 攻略 在 CentOS 下搭建 PPTP
tags:
  - CentOS
  - PPTP
  - VPN
  - VPS
  - Linux
permalink: pptp
id: 14
updated: '2016-06-04 14:20:34'
date: 2016-06-04 12:56:44
---

### 验证 ppp
用cat命令检查是否开启ppp，一般服务器都是开启的。

```
cat /dev/ppp
cat: /dev/ppp: No such device or address
```

出现这个结果，说明 ppp 没有安装

### 安装 PPP

```
yum update
yum -y install ppp iptables
```
安装 iptables 是为了做 NAT，让 PPTP 客户端能够通过 PPTP 服务器上外网，不过一般情况下 iptables 默认都是系统装好后就已经有了。

我们来看一下安装的 ppp 是什么版本

![2016-06-04_12:16:07.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-04_12:16:07.jpg-mark)

可以看到是 ppp 版本是 2.4.5-10.el6
在网站 [sourceforge](http://poptop.sourceforge.net/yum/stable/packages/) 上有所有版本的 PPTP，选择对应的版本下载。

```
cd \tmp
wget http://poptop.sourceforge.net/yum/stable/packages/pptpd-1.4.0-1.el6.x86_64.rpm
rpm -ivh pptpd-1.4.0-1.el6.x86_64.rpm
```

安装到这就完了。

### 配置 PPTP

##### 1、 添加 服务端和客户端 IP

```
vi /etc/pptpd.conf
```

在最底下添加这两行

```
localip 10.0.0.1
remoteip 10.0.0.2-10
```

localip 是 pptp 服务器端 IP

remoteip 是客户端获取的 IP 地址范围，上面是分配了 10.0.0.2 ~ 10.0.0.10 这个范围

##### 2、 添加 DNS 服务器

```
vi /etc/ppp/options.pptpd
```

在最后添加这三行，你也可以改成自己想要的 DNS 服务器

```
# DNS
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

##### 3、 添加用户

```
vi /etc/ppp/chap-secrets
```

client 填用户名，server 填 pptpd 或者 *，secret 填用户密码，IP 填给用户分配的 IP 可以指定 IP，不指定的填 *

```
# Secrets for authentication using CHAP
# client        server          secret                  IP addresses
vpn_1           pptpd           password                *
vpn_2           pptpd           password                10.0.0.2
```

##### 4、 开启 IP 转发

```
vi /etc/sysctl.conf
```

把 `net.ipv4.ip_forward` 改为 `1`

![2016-06-04_12:36:08.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-04_12:36:08.jpg-mark)

执行下面命令使内核配置生效

```
sysctl -p
```

要是出现这种情况，不用管可以忽略

```
error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key
error: "net.bridge.bridge-nf-call-iptables" is an unknown key
error: "net.bridge.bridge-nf-call-arptables" is an unknown key
```

##### 5、 配置 iptables 转发

```
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j SNAT --to-source server_ip
```

这条命令是把 `10.0.0.0` 这个网段的 IP 通过你的服务器公网 IP 访问外网

`10.0.0.0` 是在第一步在 `/etc/pptpd.conf` 设置的 `remoteip`，如果你的设置跟教程里不一样，请改成你自己设置的 IP 段

`server_ip` 改成你自己的服务器 IP

保存 iptables 配置

```
service iptables save
```

#### 6、启动服务

```
service pptpd start
service iptables start
```

设置为开机启动

```
chkconfig pptpd on
chkconfig iptables on
```

--

接下来你就可以通过 VPN 采用 PPTP 连接方式 F*Q 了

文章出自 [@laizw.cn](http://laizw.cn)
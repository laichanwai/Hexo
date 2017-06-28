---
title: 最简单最快速的方式在 CentOS 下搭建 shadowsocks
tags:
  - CentOS
  - VPN
  - VPS
  - Linux
  - shadowsocks
permalink: shadowsocks
id: 15
updated: '2016-06-04 14:19:13'
date: 2016-06-04 13:03:52
---

说到最简单最快速的方式搭建 shadowsock，那就当然是用一键脚本

接下来我们用一键脚本来快速搭建 shadowsock

### 开始安装

```
cd /tmp
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev.sh
chmod +x shadowsocks-libev.sh
./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log
```
设置密码：默认 teddysun.com

![2016-06-04_11:53:57.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-04_11:53:57.jpg-mark)

设置端口：默认 8989

![2016-06-04_11:54:51.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-04_11:54:51.jpg-mark)

然后按任意键开始安装，接下来就是等它安装完

安装完之后会看到这个界面

```
Congratulations, shadowsocks-libev install completed!
Your Server IP:your_server_ip
Your Server Port:your_server_port
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:https://teddysun.com/357.html
Enjoy it!
```

查看是否已经安装成功

```
ps -ef | grep ss-server | grep -v ps | grep -v grep
```

### 卸载命令

```
cd /tmp  # 进入 shadowsocks-libev.sh 所在目录
./shadowsocks-libev.sh uninstall
```

### 其他命令

```
service shadowsocks start   # 启动
service shadowsocks stop    # 停止
service shadowsocks restart # 重启
```

### 如果以后你需要更改 shadowsock 的配置

```
vi  /etc/shadowsocks-libev/config.json
```

支持 IPv4 与 IPv6

```
{
    "server":["[::0]","0.0.0.0"],
    "server_port":your_server_port,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"your_password",
    "timeout":600,
    "method":"aes-256-cfb"
}
```


感谢 [秋水逸冰 CentOS下shadowsocks-libev一键安装脚本](https://teddysun.com/357.html)

文章出自 [@laizw.cn](http://laizw.cn)
---
title: Ghost 折腾记一之 Ghost 搭建
tags:
  - Node.js
  - CentOS
  - Linux
  - ghost
  - Nginx
  - MySQL
permalink: ghost-build
id: 10
updated: '2016-06-17 10:53:24'
date: 2016-06-02 02:58:02
---

## 前序

接触 Ghost 之前，我曾经搞过 Hexo，感觉 Hexo 足够满足大部分人的需要，相比 Ghost 来说， Hexo 成本更低，步骤更简单，有兴趣的可以去看看我的这篇文章 [在 GitHub 上搭建 Hexo 个人博客](http://laizw.cn/hexo-blog/)

不过，对于我们这类爱折腾的人来说，Ghost 相比来说更有可玩性，更具诱惑力，把持不住的我，就这么投入了 Ghost 的怀抱。

## 步骤

1. 购买云主机

2. 在主机搭建 Node.js环境

3. 搭建 Nginx 环境

4. 安装 Ghost

5. 设置 Nginx 代理

---

### 一、购买主机

你要是不打算购买主机，你可以在 ghost 上申请个管理你的博客，地址在这：[https://ghost.org](https://ghost.org/) 

关于主机的设置等我就不在这里多说，大家可以到主机的官网上查看

### 二、在主机搭建 Node.js 环境

在这之前你得先知道 ghost 支持 Node.js 的版本

![2016-06-03_15:47:32.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-03_15:47:32.jpg-mark)

目前 Node.js 官网上最新版本是 v4.4.5 稳定版 和 v6.2.1 的开发版，也就是说，我们使用 Node.js 最新的稳定版，ghost 是支持的。

``` shell
cd /tmp
wget https://nodejs.org/dist/v4.4.5/node-v4.4.5-linux-x86.tar.xz
xz -d node-v4.4.5-linux-x86.tar.xz
tar xvf node-v4.4.5-linux-x86.tar
mv node-v4.4.5-linux-x86 /usr/local/node
```

配置环境

```
vi /etc/profile
```

在最下面添加

```
export NODE_HOME=/usr/local/node
export PATH=$NODE_HOME/bin:$PATH
```

输入下面的命令让改变生效

```
source /etc/profile
```

通过以下命令检测是否安装完成

```
node -v
npm -v
```

### 三、搭建 Nginx 环境

安装 Nginx 我们通过 yum 来安装，但是我们要先给 yum 添加 Nginx 的 repo

``` shell
vi /etc/yum.repos.d/nginx.repo
```

复制下面的到 nginx.repo 里面

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

![2016-06-06_20:00:47.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_20:00:47.jpg-mark)

然后我们就可以通过 `yum -y install nginx` 来安装 nginx

常用命令

```shell
nginx            # 启动
nginx -s stop    # 关闭
nginx -s reload  # 重启
```

启动 nginx 之后我们就能访问到 nginx 的欢迎页面

![2016-06-06_20:05:35.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_20:05:35.jpg-mark)

如果访问不了，那么得配置一下防火墙(一般都没问题)

```
iptables -I INPUT 5 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
service iptables save
service iptables restart
```

### 四、安装 Ghost

下载并解压 ghost

```
cd /usr/local
curl -L https://ghost.org/zip/ghost-latest.zip -o ghost.zip
unzip -uo ghost.zip -d ghost
cd ghost
```

配置 config.js

```
cp config.example.js config.js
vi config.js
```

默认使用 sqlite3 只需要将 production 中的 url 改了就行

![2016-06-06_20:21:02.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_20:21:02.jpg-mark)

安装 ghost 依赖 module

这里得注意一下，因为 ghost 中需要用到 sqlite3 的 module，但是这个得翻墙下载，如果你不能翻墙的话，可以安转 mysql，因为 ghost 也支持 mysql

- sqlite3

```
npm install --production
```

安装 sqlite3 失败

![2016-06-06_20:17:42.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_20:17:42.jpg-mark)


使用 sqlite3 的跳过下面这个步骤

- mysql

安装 mysql (yum 安装的 mysql 版本比较低，有需要的自己安装别的版本)

```
yum -y install mysql-server mysql mysql-devel
```

启动 mysql

```
service mysqld start
```

更改 mysql 默认密码

```
mysqladmin -uroot -p password your_pass
```

mysql 默认密码为空，直接回车就行

新建数据库 ghost

```
mysql -uroot -p
mysql > create database ghost;
mysql > exit
```

配置 config.js

![2016-06-06_20:37:05.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_20:37:05.jpg-mark)

修改 package.json 找到所有 `"sqlite3": "3.1.3"` 并删掉，只要看到 `sqlite3` 就删掉

然后执行下面命令安装依赖 module

```
npm install --production
```

安装完运行 ghost

```
npm index --production
```

用下面这个命令可以使 ghost 退出终端保持运行，你也可以用 ghost 上推荐的方式 [让 Ghost 一直运行](http://docs.ghost.org/zh/installation/deploy/)

```
nohup npm index --production &
```

停止 ghost

```
lsof -i :2368
kill ghost_pid # ghost_pid 是上面那条命令里得到的 ghost 的 pid
```

### 五、配置 Nginx 代理

```
cd /etc/nginx/conf.d/
vi default.conf
``` 

把 `server_name` 改为自己的域名

在 `location / {...}` 里面添加 `proxy_pass   http://127.0.0.1:2368;`

![2016-06-06_21:05:21.jpg](http://7xlykq.com1.z0.glb.clouddn.com/image/2016-06-06_21:05:21.jpg-mark)

重启 nginx

```
service nginx restart
```

## END

接下来就可以访问你的域名来访问 ghost 
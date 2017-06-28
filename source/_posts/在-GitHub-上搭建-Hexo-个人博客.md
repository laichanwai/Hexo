---
title: 在 GitHub 上搭建 Hexo 个人博客
tags:
  - Blog
  - Node.js
  - Hexo
permalink: hexo-blog
id: 6
updated: '2016-05-26 13:38:14'
date: 2016-05-26 13:31:30
---

## Hexo 简介

[Hexo](https://github.com/hexojs/hexo) 是一款基于 Node.js 的静态博客框架。

***用 Hexo 搭建博客有很多好处:***

- 首先它支持 Markdown，这样我们撰写博客就会变得十分方便。

- Hexo 的扩展性也十分强大，它支持EJS、Swig，我们还可以通过插件让它支持Haml，Jade，Less等。

- Hexo 支持多线程，生成几百篇文章所需要的时间只不过几秒钟。

- Hexo 支持一键部署，只需要一条命令就可以把博客发布到 GitHub 上。

<!-- more -->
---
## 废话不多说，直接上代码

因为 Hexo 是基于 Node.js 的，所以在开始 Hexo 之前要在机器上搭建好 Node.js 的环境，那在这里我就不讲解 Node.js 的搭建了，大家可以在[Node.js 中文网](http://nodejs.cn/)([英文](https://nodejs.org/en/)) 上看看。


### 安装 Hexo ( -g 代表把 Hexo 全局安装到系统中)
``` bash
$ npm install hexo -g
```

### 升级到最新版(这个可选如果初次安装一般都是最新版)
``` bash
$ npm update hexo -g
```
这样子 Hexo 就安装完了，简单吧~

### 接下来我们开始新建一个项目
``` bash
$ cd ~/Documents
$ hexo init hexoBlog
```
`hexoBlog` 是自己取的一个名字。

这行代码的作用是在 `Documents` 目录下自动新建一个名为 `hexoBlog` 文件夹，Hexo 会在这个文件夹内自动生成资料文件。

**来看看这个文件夹有什么:**
![hexoBlog](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-init.png)


### 安装 Hexo 需要的模块
``` bash
$ cd hexoBlog
$ npm install
```

接下来 npm 会自动帮我们安装所需要的模块，静静的等它安装完，然后我们看看安装完的文件夹变化
![hexoBlog](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-new.png)

这时候多了几个文件夹，这里我先不解释他们的作用，晚点我会再写一篇博客具体讲解里面的所有文件。

**到这一步博客就已经安装完了**

### 新建博客

-  我们来新建一篇博客

``` bash
$ hexo new "hello hexo" # 等同于 hexo n
```
`hello hexo` 是我们新建博客标题。

- 生成静态网站

``` bash
$ hexo generate # 等同于 hexo g
```
Hexo 速度很快，几乎秒生成。

- 在本地跑一下看看效果

``` bash
$ hexo server # 等同于 hexo s
```
Hexo 默认 ***4000*** 端口，打开浏览器输入 ***http://localhost:4000*** 看看我们的博客。

![hexoBlog-Site](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-site.png)

可以看到我们新建的那篇 ***hello hexo***


### 把我们的博客放到 GitHub 上

#### 首先我们要知道一些关于 GitHub Pages:

1. GitHub Pages 是免费的静态站点;
2. 搭建简单而且免费;
3. 自带主题，支持自定义页面;
4. 支持绑定域名;

我们可以看到，GitHub Pages 十分适合我们在上面挂我们的博客。

#### 在 GitHub 上搭建我们的博客有两种方式：

- 为个人或组织建立 Pages。
- 为某个项目建立 Pages。

上面两种方式我都会给大家详细讲解一下。

##### 为个人或组织建立 Pages (以我的为例)
我在 GitHub 上的用户名是`laichanwai`

1. 登录 GitHub 并在 自己或组织 下建立一个名为 `laichanwai.github.io` 的仓库。![我的 Pages 仓库](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-github.png)

2. 完成了，我们可以通过  [http://laichanwai.github.io](http://laichanwai.github.io/) 来访问我们的博客。

##### 为某个项目建立 Pages

1. 登录 GitHub，进入具体的项目
2. 新建一个名为 `gh-pages` 的 Branch(分支)，注意名字一定要输入正确。![项目 Pages](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-github-project.png)
3. 完成了，我们可以通过 *http://username.github.com/projectname* 访问我们的站点。

### 在 Hexo 上配置发布到 GitHub

在本地进入到 `hexoBlog` 目录，然后用编辑器打开 `_config.yml` 这个文件。

![_config.yml](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-config-deploy.png)

注意到最下面的那两行

```
deploy:
  type:
```

我们把它改为

![deploy](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-config-deploy-new.png)

`repo:` 的地址在 GitHub 上获取 (_假设你已经在 GitHub 上添加了本机的 SSH_)。
![SSH](http://7xlykq.com1.z0.glb.clouddn.com/hexoblog-github-ssh.png)

### 接下来把博客 deploy 到 GitHub 上

切换到终端输入:

``` bash
$ hexo deploy # 等同于 hexo d
```

#### 大功告成啦!

### 总结一下

#### 搭建 Hexo

``` bash
$ npm install hexo -g		# 安装 Hexo
$ hexo init hexoBlog		# 新建 Hexo 项目 <hexoBlog>
$ cd hexoBlog			# 进入 hexoBlog
$ npm install			# 安装 Hexo 模块
$ hexo new "Hello Hexo"		# 新建一篇博客
$ hexo generate		# 生成静态网站
$ hexo server			# 在本地跑一遍网站 默认 http://localhost:4000
$ hexo deploy			# 开始部署
```

#### 更新博客

以后写博客的时候只需要做下面几个操作就可以了，十分方便。

``` bash
$ hexo new "title"	# 新建博客
```
``` bash
$ hexo g	# 生成静态网站
```
``` bash
$ hexo s	# 在本地预览网站效果
```
``` bash
$ hexo d	# 开始部署
```

***
我是华丽的分界线
---

> 这是小弟第一次写博客，可能有很多不足的地方，希望大家可以多多指教。

> 我是一个iOS码农，刚接触前端不到一个星期，如果博客要是有错漏的地方希望大家能够给我指出，谢谢各路大侠。

> 接下来我会和大家一起学习，一边学习一边完善我的Blog。

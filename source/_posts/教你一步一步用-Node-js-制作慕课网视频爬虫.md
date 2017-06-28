---
title: 教你一步一步用 Node.js 制作慕课网视频爬虫
tags:
  - Blog
  - Node.js
permalink: imooc-spider
id: 2
updated: '2016-05-24 14:26:01'
date: 2016-05-23 14:46:11
---

## 开始

这个教程十分适合初学 Node.js 的初学者看(因为我也是一只初学的菜鸟~)

在这里，我就默认大家都已经在自己的电脑上搭建好 Node.js，我就不再多讲了，如果你是第一次接触 Node.js 那么先请到可以到[Node.js 中文网](http://nodejs.cn/)([英文](https://nodejs.org/en/)) 上看看，里面有完整的安装教程。

想直接看源码的可以直接移步到 github [imooc-video-download](https://github.com/laichanwai/imooc-video-download)。

## 第一步

说到下载视频，首先我们要先有个大概思路：

- **发送请求—>获取视频下载地址->发送下载请求下载视频—>把视频文件写入本地**

1. 因为需要发起请求，在这里我用到  [superagent ](http://visionmedia.github.io/superagent/ ) 是个 http 方面的库，可以发起 get 或 post 请求。

2. 因为要解析网页，在这里我使用  [cheerio ](https://github.com/cheeriojs/cheerio ) 我们可以把它当做一个 Node.js 版的 jquery，用来从网页中以 css selector 取数据，使用方式跟 jquery 一样的。

3. 写文件的话就用 Node.js 自带的 `fs` 文件模块。

## 第二步

既然思路有了，那就开始上代码吧~

### 新建一个项目:

``` bash
$ cd ~/Documents/NodeJs
$ mkdir immoc-video-download
$ cd immoc-video-download
$ npm init
```
ps：接下来会让你填很多信息，全部直接回车就可以（也可以认真写写）,最后需要输入 `yes` 加回车结束。

我解释一下一步，`npm init` 是初始化一个项目，互动式帮我们生成一个最简单的 package.json 文件，而这个 package.json 文件包含我们项目名，作者，项目依赖等等信息。

### 安装项目模块
``` bash
$ npm install superagent cheerio --save
```

`--save` 是个可选参数，加了它之后会自动帮我们把上面安装的两个模块自动写入到 package.json 里面。

### 新建入口文件

``` bash
$ touch app.js #新建一个名为 app.js 的文件
```
到这一步，这个爬虫项目所需要的环境和依赖的都已经准备好了。

* * *

## 第三步

### 引入模块依赖:

用编辑器打开 *app.js* 这个文件，在里面输入

``` javascript
var superagent = require('superagent');
var cheerio = require('cheerio');
var fs = require('fs');
```

### 搭好基本框架

``` javascript
var superagent = require('superagent');
var cheerio = require('cheerio');
var fs = require('fs');

var url = '慕课网课程'
var savePath = '保存文件路径'

superagent
    .get(url)
    .end(function(err, res) {
        
    })

// 获取视频 url
var getVideoUrl = function() {
    
}

// 下载视频
var downloadVideo = function() {
    
}
```

基本框架已经没问题了，不过里面啥都没，怎么办？

别慌，爬虫嘛，当然是在网页上爬数据。

## 第四步

### 分析慕课网

#### 源码解析

**先上图！**
[http://www.imooc.com/learn/441](http://www.imooc.com/learn/441)

![imooc 源码](http://upload-images.jianshu.io/upload_images/1271031-70e69f952aa87f22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- ***看这里！！***

1. 我们来看看慕课网的网站源码，发现没？每个视频都有一个视频的id，再来看看课程的 *URL* 课程也有一个课程 ID 。

2. 我们都知道这个网站的视频是需要登录之后才可以看观看的，这就代表我们在请求的时候要在 headers 里做些手脚，把 cookies 放进去


#### 请求头分析

**先上图！**

![imooc headers](http://upload-images.jianshu.io/upload_images/1271031-6c0883e5bf4c9bd1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看完这些我们应该有个大概清晰的思路了吧，废话不多说，完善代码。

### 解析视频 id 和 filename

#### 看看核心代码

##### 先写一个请求头：

``` javascript
var headers = {
    "Cache-Control": "max-age=0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Referer": "http://www.imooc.com/",
    "Accept-Encoding": "gzip, deflate, sdch",
    "Accept-Language": "zh-CN,zh;q=0.8",
    "Cookie": cookies // 在外面定义一个 cookies 变量存放自己的 cookies
};
```

这里就是直接把上一个截图里的 request headers 里面的所有数据都 copy 进去。

##### 解析 html：

``` javascript
superagent
    .get('www.imooc.com/learn/' + courseId) // 在外面设置一个 courseId 的参数
    .set(headers)
    .end(function(err, res) {
        
        // res.text 通过请求获取的 html 页面
        var $ = cheerio.load(res.text);
        
        // 获取课程的名称
        $('.course-infos .hd').find('h2').each(function(item) {
            courseTitle = $(this).text();
        })

        // .chapter 是包含所有 video 的容器，这是 jquery 语法，为了获取所有的视频 id 和 filename
        $('.chapter').each(function(item) {

            var videos = $(this).find('.video').children('li')

            videos.each(function(item) {
                var video = $(this).find('a')
                var filename = video.text().replace(/(^\\\\\\\\s+)|(\\\\\\\\s+$)/g,"");
                var id = video.attr('href').split('video/')[1]
                
                // 视频 id 和 视频文件名字
                console.log(id， filename);
            })
        })
    })
```

上面那段代码帮我们获取了每个视频的 id 和 它的文件名，那么接下来我们只需要获取它的视频下载地址就 ok。

##### 获取下载地址：

这时候我们还得到视频播放页面去抓取一下网站播放视频请求的地址：

![imooc headers](http://upload-images.jianshu.io/upload_images/1271031-a7991d3a39b42a5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在过滤器中输入视频的格式 mp4 过滤一下(找不到的话试试 flv 之类的)，看到出现了我们想要的 mp4 文件，是不是特别激动！！

看一下头文件，什么鬼？！这串东西什么？？可以肯定，这不是我们想要的。

*我们试试输入视频 id 作过滤条件*

![imooc headers](http://upload-images.jianshu.io/upload_images/1271031-3065aa65368adc48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*点那个 preview(response显示的东西太长很难截图) 看看它给我们返回什么：*
![imooc headers](http://upload-images.jianshu.io/upload_images/1271031-bc596d03332eb86e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看！出来了。


我们可以看到这是 ajax 请求的一个地址，没关系，既然给我们找到了，那就拼接一下就 ok 了。

上代码！

``` javascript
var getVideoUrl = function(id, callback) {
    superagent.get('http://www.imooc.com/course/ajaxmediainfo/?mid=' + id + '&mode=flash')
        .end(function(err, res) {
            var url = JSON.parse(res.text);

            if(url.result == 0) {
                url = url.data.result.mpath[0];
                callback(url);
            }
        })
}
```

##### 下载视频

有了上面的 url 接下来我们的功夫就简单多了。

直接上代码：

``` javascript
var downloadVideo = function(url, filename, callback) {

    // 去掉文件名后面的时间
    // 2-1 登录动画-冒泡 (10:53) —> 2-1 登录动画-冒泡.mp4
    filename = filename.replace(/\\\\\\\\(.*\\\\\\\\)/,'') + '.mp4';

    // 创建一个以课程名字命名的目录存放视频
    var dirPath = savePath + courseTitle + '/'
    if (!fs.existsSync(dirPath)) {
        fs.mkdirSync(dirPath);
    }

    console.log('开始下载第' + courseTotalCount + '个视频' + filename + ' 地址: ' + url);
    var writeStream = fs.createWriteStream(dirPath + filename);
    writeStream.on('close', function() {

        callback(filename);
    })

    var req = superagent.get(url)
    req.pipe(writeStream);

}
```

看到这里是不是特别兴奋！！！

别急，接着我们再加入这行代码。

``` javascript
var courseId = process.argv.splice(2, 1);
```

ok！大功告成！

## 第五步

我们下载视频的时候只需要在终端执行下面这行命令就可以了。

``` bash
$ node app courseId # 你想下载的视频 id
```

这个项目已上传到 github [imooc-video-download](https://github.com/laichanwai/imooc-video-download)
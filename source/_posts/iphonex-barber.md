---
title: 我专门为了 iPhone X 开了家理发店
tags:
  - Blog
  - iOS
date: 2017-09-25 18:41:03
permalink: iphonex-barber
---

> 向巨丑的苹果 “致敬”

> 我开了家理发店，客户只针对 iPhone X，由于资金有限，目前店内只有一个专业理发师 Tony。

如果您的 iPhone X 需要理发，请按照以下步骤预约：

1. 通过 cocoapods 集成 [Barber](https://github.com/laichanwai/Barber)

    ```
    pod 'Barber'
    ```

2. 为了更好的让 Tony 设计你的 iPhone X 的头发，请在 info.plist 里面添加以下这句话

    ```xml
    <key>UIViewControllerBasedStatusBarAppearance</key>
    <false/>
    ```
    
3. 在合适的时间过来找 Tony 剪发

    ![](http://upload-images.jianshu.io/upload_images/1271031-39d5019b1a2a7183.png-watermark?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 选择合适的发型

    Tony 不才，暂时只会两种手法：平头和圆脸
    
    ```objc
    HairTypeFlatTop,    // 平头
    HairTypeRoundFace,  // 圆脸
    ```

    - 平头
    
        ![20170925150630900976032.png](http://storage.laizw.cn/images/upi/20170925150630900976032.png-500x500)

    - 圆脸
    
        ![20170925150630905072718.png](http://storage.laizw.cn/images/upi/20170925150630905072718.png-500x500)


如果各位有兴趣，可以加盟我的理发店：[Barber](https://github.com/laichanwai/Barber)

> 再次向巨丑的苹果 “致敬”
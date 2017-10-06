---
layout: post
title: WebClip中设置添加至主屏幕的图标
author: kinshines
date:   2016-12-24
categories: html
permalink: /archivers/webclip-webpage-icon
---

<p class="lead">Web应用作为仿原生应用的重要补充，我们希望添加在主屏幕时，也能有像原生应用那样逼真的icon，本文介绍关于webclip的icon设置</p>

* 对于全局网站设置icon，将png格式的图片文件命名为 apple-touch-icon.png 并置于根目录，无需其他配置
* 对于单个网页设置icon，在head中加入link

        <link rel="apple-touch-icon" href="/custom_icon.png">

* 对于不同分辨率的设备分别设置不同的icon，可以使用size属性

        <link rel="apple-touch-icon" href="touch-icon-iphone.png">
        <link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad.png">
        <link rel="apple-touch-icon" sizes="180x180" href="touch-icon-iphone-retina.png">
        <link rel="apple-touch-icon" sizes="167x167" href="touch-icon-ipad-retina.png">


扩展阅读：

[Configuring Web Applications](https://developer.apple.com/library/content/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html#//apple_ref/doc/uid/TP40002051-CH3-SW4)

[Configuring the Viewport](https://developer.apple.com/library/content/documentation/AppleApplications/Reference/SafariWebContent/UsingtheViewport/UsingtheViewport.html#//apple_ref/doc/uid/TP40006509-SW19)


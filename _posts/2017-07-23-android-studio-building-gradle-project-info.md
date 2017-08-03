---
layout: post
title: AndroidStudio首次新建项目一直卡在Building gradle project info解决方案
author: kinshines
date:   2017-07-23
categories: android
permalink: /archivers/android-studio-building-gradle-project-info-waiting
---

今天安装AndroidStudio，首次新建项目时，界面一直卡在Building gradle project info的进度条不动，实际上是因为首次新建项目，需要下载该项目需要的gradle版本，这里具体需要下载的是gradle-3.3-all.zip，大小是87.1MB。不知是被墙了还是什么原因，下载速度非常慢(<10k/s)， 最终导致一直卡住，直至下载完成(通常由于网络波动等原因导致下载失败)


索性直接去官网[Gradle Distributions](http://services.gradle.org/distributions/)上，找到对应的版本下载地址，使用迅雷几秒钟下载完成后，将zip压缩包直接放到C:\Users\Admin\\.gradle\wrapper\dists\gradle-3.3-all目录下。重启AndroidStudio，新建项目时，发现它在上面提到的目录下创建了一个名为55gk2rcmfc6p2dg9u9ohc3hw9的文件夹，仍然去下载gradle-3.3-all.zip 并在该目录下生成了.part和.lock文件

于是又将gradle-3.3-all.zip文件移动到这个55gk2rcmfc6p2dg9u9ohc3hw9文件夹下(怀疑命名是随机的，在其他电脑或版本上不一定要是叫这个名字)，并删除gradle-3.3-all.zip.part。再次重启AndroidStudio，新建项目，发现AndroidStudio可以识别到我放置的gradle-3.3-all.zip文件，并自行解压，问题解决。


其实这个问题并不是什么大问题，但是仍然值得记录下来，因为万事开头难，当我们学习一门陌生的新语言时，语法什么的其实并不难掌握，最难把控的是入门的开发环境搭建、配套工具的使用以及对这门语言生态的理解，刚开始遇到的这种挫折极容易打击学习的积极性，让初学者望而却步，只可远观而不敢近玩，停滞不前，这才是最可怕的。

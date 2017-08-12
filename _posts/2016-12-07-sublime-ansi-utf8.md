---
layout: post
title: Sublime编辑器中ANSI编码的汉字出现乱码解决方法
author: kinshines
date:   2016-12-07
categories: develop
permalink: /archivers/sublime-ansi-utf8
---

<p class="lead"> 使用sublime打开一个ANSI编码的文件,会出现乱码，本文将介绍使用ConvertToUTF8插件的方式解决此问题</p>

### Package Control

安装插件需要用到 Package Control ，如果sublime尚未安装Package Control，需要先安装，参考[Sublime Package Control 的安装](https://kinshines.github.io/archivers/sublime-package-control)


### 安装 ConvertToUTF8 插件

快捷键 Ctrl+Shift+P（菜单 – Tools – Command Paletter），输入 install 选中Install Package并回车，输入或选择需要的插件: ConvertToUTF8 回车就安装了（注意左下角的小文字变化，会提示安装成功）


关闭Sublime后，重新打开那个ANSI编码的文件，就会发现没有乱码了
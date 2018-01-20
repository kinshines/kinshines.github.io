---
layout: post
title: .NET Core 2.0 发送 Email
author: kinshines
date:   2017-12-20
categories: develop
permalink: /archivers/vscode-teminal-fontface
---

<p class="lead">使用VS Code 编写Angular项目时，因为将编辑器字体设置为Segoe UI 和微软雅黑，导致终端显示的字体非常宽，并且不能换行，导致输入命令和查看输出时很不方便。经查得知，终端的字体也是可以单独设置的，下面记录具体设置过程</p>

首先，我将编辑器字体如下设置：
        "editor.fontFamily": "'Segoe UI', 'Microsoft YaHei'",
        "editor.fontSize": 18,

设置完成后，编辑器模式显示如我所愿，但是终端显示就不正常了，如下图：
![wide terminal](https://user-images.githubusercontent.com/7374965/33870655-cc5ca560-df49-11e7-8820-aa60e943ed6e.png)

于是，继续设置终端字体如下：
        "terminal.integrated.fontFamily": "Consolas,'Courier New',monospace",
        
显示正常，问题解决
---
layout: post
title: cmd命令行窗口使用Consolas字体
author: kinshines
date:   2016-09-29
categories: develop
permalink: /archivers/cmd-consolas-font
---

<p class="lead">对于经常使用cmd窗口的朋友们，或许早已厌烦了cmd窗口的点阵字体了吧，程序员们敲代码本身已经够辛苦的了，当然要努力改善编码环境，下面就将cmd窗口的点阵字体修改成程序员专用字体Consolas</p>

### 代码页
打开cmd窗口，在标题栏右键属性，切换到字体选项卡，只有点阵字体和新宋体两个选项，并没有其他的可选字体，这个问题源于Windows对Console程序的设定，打开注册表定位至[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Console\TrueTypeFont]，就可以看到Windows下Console程序的TrueType字体设定了

![ConsoleFont](https://kinshines.github.io/img/develop/cmd-font_1.png)

可以看到936等几个字符串值每一项都对应一个代码页，比如936对应简体中文，950代表繁体中文等。0和00（000是我后加的）两项则比较特殊，其实这两个都是代码页437对应的字体。也就是说除了代码页437之外，其他的代码页只能指定一种可用的字体，否则就要使用点阵字体

### 解决方案
因此，可以很简单的将代码页切换到437，然后选择喜欢的字体即可。当然了默认情况下代码页437可用的字体也只有Lucida Console和Consolas，切换代码页命令：

        chcp 437

这时，再回到属性页的字体选项卡，就可以选择Consolas字体了

![ConsolasFont](https://kinshines.github.io/img/develop/cmd-font_2.png)


### 更多字体
如果对Consolas在命令行程序中的效果不满意的话，就要考虑一下增加命令行程序可用的字体了。要增加字体其实也很简单，在注册表中[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Console\TrueTypeFont]项下增加一项名字为000的字符串值，并将其值设置为你想要用的字体的名字，如果要增加更多字体只要再增加一项字符串值，并将其命名为0000，也就是多加一个0即可。下图中我增加了一个名为Bitstream Vera Sans Mono的字体，你可以选择任何你想用的字体，当然了必须是系统中已经有的
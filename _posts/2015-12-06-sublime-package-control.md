---
layout: post
title: Sublime text 2/3 中 Package Control 的安装与使用方法
author: kinshines
date:   2015-12-06
categories: develop
permalink: /archivers/sublime-package-control
---

<p class="lead"> Package Control 插件是一个方便 Sublime text 管理插件的插件，但因为 Sublime Text 3 更新了 Python 的函数，API不同了，导致基于 Python 开发的插件很多都不能工作，Package Control 原来的安装方法都失效了。此处将 wbond 网站的 ST3 Package Control 简便安装方法翻译转至此处，方便查阅</p>

### 简单的安装方法

从菜单 View - Show Console 或者 ctrl + ~ 快捷键，调出 console。将以下 Python 代码粘贴进去并 enter 执行，不出意外即完成安装。以下提供 ST3 和 ST2 的安装代码：

Sublime Text 3：

        import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())

Sublime Text 2：

        import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')

### 手动安装

可能由于各种原因，无法使用代码安装，那可以通过以下步骤手动安装Package Control：

1. 点击Preferences > Browse Packages菜单
2. 进入打开的目录的上层目录，然后再进入Installed Packages/目录
3. 下载 [Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package) 并复制到Installed Packages/目录
4. 重启Sublime Text。

### 使用方法

快捷键 Ctrl+Shift+P（菜单 – Tools – Command Paletter），输入 install 选中Install Package并回车，输入或选择你需要的插件回车就安装了（注意左下角的小文字变化，会提示安装成功）。

![package_control2](https://www.jeffdesign.net/wp-content/uploads/2014/05/sublime_package_control2.png)
![package_control3](https://www.jeffdesign.net/wp-content/uploads/2014/05/sublime_package_control3.png)
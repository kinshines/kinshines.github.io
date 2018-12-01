---
layout: post
title: 为 windows cmd 设置上网代理
author: kinshines
date:   2018-12-01
categories: develop
permalink: /archivers/windows-cmd-http-proxy
---

<p class="lead">最近涉及到nodejs开发，需要在命令行模式下下载package，但是某些国外站点的包却下载不了，只能使用科学上网了，本文记录如何在命令行模式下，翻墙下载文件。</p>

ShadowSocks 这类工具尽管开启了全局代理，但是cmd里依旧无法下载成功。

这种全局代理只针对使用IE代理的程序才全局，不是像VPN那样的全局。当然也更不支持PAC模式了。

cmd如果要设置代理的话，需要在执行其他命令之前，先执行一下

    set http_proxy=http://127.0.0.1:1189
    set https_proxy=http://127.0.0.1:1189

(上面代理地址只是示例，请换成你自己的代理地址)

上面命令的作用是设置环境变量，不用担心，这种环境变量只会持续到cmd窗口关闭，不是系统环境变量。

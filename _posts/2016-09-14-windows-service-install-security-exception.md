---
layout: post
title: Windows服务安装异常：System.Security.SecurityException
author: kinshines
date:   2016-09-14
categories: develop
permalink: /archivers/windows-sevice-install-security-exception
---

<p class="lead">在Windows10 上，安装Windows服务时出现安装异常：System.Security.SecurityException: 未找到源，但未能搜索某些或全部事件日志。不可 访问的日志: Security</p>

两种方法处理：
1. 在安装bat文件上右键，选择“以管理员身份运行”
2. 以管理员身份运行VisualStudio，然后打开项目，再执行InstallService程序安装服务
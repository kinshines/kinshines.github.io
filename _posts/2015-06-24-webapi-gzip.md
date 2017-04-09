---
layout: post
title: ASP.NET Web API 使用ActionFilter实现 GZip compression
author: kinshines
date:   2015-06-24
categories: webapi
permalink: /archivers/WebAPI-GZip-Compression
---


<p class="lead">本文演示WebAPI 开启 GZip / Deflate 压缩以提高性能</p>
压缩是一种缩小传输包大小的简单方法，从而提高客户端和服务器的传输效率。
当前Web传输中有两种通用的成熟压缩算法：GZip 和 Deflate，这两种算法都能被浏览器识别而且在HTTP的响应中可以自动解压缩。
下图展示了GZip压缩效果：
![Compression](http://blog.developers.ba/wp-content/uploads/2014/06/compression.png "Compression")
图片来源：<a href="http://www.sendung.de/2007-04-09/web-services-output-formats-and-gzip-compression/">Effects of GZip compression</a>
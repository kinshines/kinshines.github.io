---
layout: post
title: ASP.NET Web API 使用ActionFilter实现 GZip compression
author: kinshines
date:   2015-06-24
categories: webapi
permalink: /archivers/WebAPI-GZip-Compression
---


<p class="lead">本文演示WebAPI 开启 GZip / Deflate 压缩以提高Web性能</p>
Web压缩是通过压缩传输包大小的方式提高客户端和服务器之前的传输效率。
当前Web传输中有两种通用的成熟压缩算法：GZip 和 Deflate，这两种算法都能被浏览器识别而且在HTTP的响应中可以自动解压缩。
下图展示了GZip压缩效果：
![Compression](http://blog.developers.ba/wp-content/uploads/2014/06/compression.png "Compression")
图片来源：<a href="http://www.sendung.de/2007-04-09/web-services-output-formats-and-gzip-compression/">Effects of GZip compression</a>

目前在ASP.NET Web API上实现压缩有已下三种方式：
1.IIS级别，在IIS中设置开启即可
2.自定义的委托Handler
3.自定义的ActionFilter，可应用在Method级别，Controller级别甚至整个WebAPI级别


## DotNetZip
下面演示ActionFilter实现GZip压缩，前提要借助第三方库<a href="http://dotnetzip.codeplex.com/">DotNetZip library</a>
Nuget获取：
{% highlight js %}

Install-Package DotNetZip

{% endhight %}
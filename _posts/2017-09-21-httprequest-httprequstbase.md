---
layout: post
title: HttpRequestBase 和 HttpRequest 之间相互转化
author: kinshines
date:   2017-09-21
categories: webapi
permalink: /archivers/httprequest-httprequestbase
---

<p class="lead">在早期的ASP.net主要是aspx和ashx中，我们常常使用HttpRequest 来获取请求参数，而在流行的WebAPI及MVC框架中，则更多的使用HttpRequestBase，本文介绍两者之间的转化，便于解决不同web框架的兼容性问题</p>


### WebAPI及MVC中获取HttpRequest

        System.Web.HttpContext.Current.Request

这里的关键是要引用正确的命名空间，才能拿到正确的HttpContext

### HttpRequest转化成HttpRequestBase

        var wrapper = new HttpRequestWrapper(httpRequest);

其中 HttpRequestWrapper 继承自 HttpRequestBase
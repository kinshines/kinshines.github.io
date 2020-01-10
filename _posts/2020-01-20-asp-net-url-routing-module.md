---
layout: post
title: 为旧版本的IIS添加web.config Routing 配置节
author: kinshines
date:   2020-01-20
categories: mvc
permalink: /archivers/asp-net-url-routing-module
---

<p class="lead">路由机制(Routing)是从ASP.NET 4.0开始引入的，对于旧版本(例如WinServer2008 R2)的IIS并不识别，需要在web站点下的web.config中稍加配置方可正常使用</p>

对于旧版本的IIS来说，url指向的是一个实实在在的文件，因此默认会去磁盘上寻找对应的文件，而新的路由机制下，并不存在这样的一个文件，因此会出现404.0-Not Found错误，如下图：
![FileResult](https://kinshines.github.io/img/mvc/notfound.png)

网上有种解决方案是web.config中配置：

{% highlight xml %}
<modules runAllManagedModulesForAllRequests="true">
{% endhighlight %}

这种方案虽然可以运行正常，但是明显加大了IIS负荷，因为一些静态文件css,js等不需要走ManagedModule，
所以此方案不可采用

正确的解决方案应该是在配置项中增加URL Routing 配置节

{% highlight xml %}

<configuration>
  <system.webServer>
    <modules>    
      <remove name="UrlRoutingModule-4.0" />
      <add name="UrlRoutingModule-4.0" type="System.Web.Routing.UrlRoutingModule" preCondition="" />
    </modules>
  </system.webServer>
</configuration>

{% endhighlight %}
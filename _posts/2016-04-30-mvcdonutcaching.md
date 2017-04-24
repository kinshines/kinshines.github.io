---
layout: post
title: 甜甜圈缓存mvcdonutcaching 的使用介绍
author: kinshines
date:   2016-04-30
categories: mvc
permalink: /archivers/mvcdonutcaching
---

<p class="lead">在Web开发中我们会遇到这样的需求：为了性能需要，网站的首页往往需要缓存，但是网页顶部的用户信息确实因人而异的，并且对用户未登录、登录后、退出后显示的内容也要适时调整，而这些内容是不能缓存的。因此今天引入一个有效也有趣的缓存策略：甜甜圈缓存-mvcdonutcaching</p>

[GitHub-mvcdonutcaching](https://github.com/moonpyk/mvcdonutcaching)

[NuGet Packages-MvcDonutCaching](https://www.nuget.org/packages/MvcDonutCaching)

### 使用方法
对于像用户登录信息这些无需缓存的内容，View中使用方法：
{% highlight java %}
@Html.Action("Login", "Account", true)
{% endhighlight %}

而对于首页这些需要缓存的内容，在Action上添加属性DonutOutputCacheAttribute：
{% highlight java %}
[DonutOutputCache(Duration = "300")]
public ActionResult Index()
{
	return View();
}
{% endhighlight %}
或者使用cache profiles:
{% highlight java %}
[DonutOutputCache(CacheProfile = "FiveMins")]
public ActionResult Index()
{
  	return View();
}
{% endhighlight %}

如果使用cache profiles，记得要在web.config中配置相关信息：
{% highlight xml %}
<caching>
  <outputCacheSettings>
    <outputCacheProfiles>
      <add name="FiveMins" duration="300" varyByParam="*" />
    </outputCacheProfiles>
  </outputCacheSettings>
</caching>
{% endhighlight %}
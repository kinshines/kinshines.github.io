---
layout: post
title: .NET Core 应用 InMemory Cache
author: kinshines
date:   2017-06-30
categories: core
permalink: /archivers/net-core-inmemory-cache
---

<p class="lead">上一节讲到<a href="https://kinshines.github.io/archivers/net-core-redis-cache">Net Core 应用 Redis</a> 在单一或小型的站点中，往往应用不到Redis这个量级的缓存服务器，这时候，一般倾向于本地的memory 缓存</p>

## MemoryCache
### startup.cs
首先，在ConfigureServices方法中增加配置：

{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
	services.AddMvc();
 
	services.AddMemoryCache();
}
{% endhighlight %}

### HomeController 

{% highlight java %}
public class HomeController : Controller
{
	private readonly IMemoryCache _memoryCache;
 
	public HomeController(IMemoryCache memoryCache)
	{
		_memoryCache = memoryCache;
	}
 
	[HttpGet]
	public string Get()
	{
		var cacheKey = "TheTime";
		DateTime existingTime;
		if (_memoryCache.TryGetValue(cacheKey, out existingTime))
		{
			return "Fetched from cache : " + existingTime.ToString();
		}
		else
		{
			existingTime = DateTime.UtcNow;
			_memoryCache.Set(cacheKey, existingTime);
			return "Added to cache : " + existingTime;
		}
	}
}
{% endhighlight %}

memory cache 还有以下几个特性：

#### 缓存过期时执行的方法 PostEvictionCallback

{% highlight java %}
_memoryCache.Set(cacheKey, existingTime, 
	new MemoryCacheEntryOptions()
		.RegisterPostEvictionCallback((key, value, reason, state) => {  /*Do Something Here */ })
 );
{% endhighlight %}

#### 缓存失效 CancellationToken ，不常用
{% highlight java %}
var cts = new CancellationTokenSource();
_memoryCache.Set(cacheKey, existingTime, 
	new MemoryCacheEntryOptions()
		.AddExpirationToken(new CancellationChangeToken(cts.Token))
 );
{% endhighlight %}

## Distributed Memory Cache
Distributed Memory Cache 可以看做是向RedisDistributedCache 的一个过渡方案，它的用法也与RedisDistributedCache 基本相同
### startup.cs
{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
	services.AddMvc();
 
	services.AddDistributedMemoryCache();
}
{% endhighlight %}

### DistributedController
{% highlight java %}
public class DistributedController : Controller
{
	private readonly IDistributedCache _distributedCache;
 
	public DistributedController(IDistributedCache distributedCache)
	{
		_distributedCache = distributedCache;
	}
 
	[HttpGet]
	public async Task<string> Get()
	{
		var cacheKey = "TheTime";
		var existingTime = _distributedCache.GetString(cacheKey);
		if (!string.IsNullOrEmpty(existingTime))
		{
			return "Fetched from cache : " + existingTime;
		}
		else
		{
			existingTime = DateTime.UtcNow.ToString();
			_distributedCache.SetString(cacheKey, existingTime);
			return "Added to cache : " + existingTime;
		}
	}
}
{% endhighlight %}

需要注意的是，IDistributedCache无法应用PostEvictionCallback 和CancellationTokens 这两个特性
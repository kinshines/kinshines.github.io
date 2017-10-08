---
layout: post
title: .NET Core 应用 Redis Cache
author: kinshines
date:   2017-06-26
categories: core
permalink: /archivers/net-core-redis-cache
---

<p class="lead">Redis作为高性能的分布式缓存，在Core里获得了微软的很大的重视。在此之前，我们需要借助第三方类库如ServiceStack.Redis 才能连接Redis，现在微软提供了默认实现</p>

### nuget
首先，要安装Package：

        Install-Package Microsoft.Extensions.Caching.Redis

### startup.cs
第二步，在ConfigureServices方法中增加配置：

{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
	services.AddMvc();
 
	services.AddDistributedRedisCache(option =>
	{
		option.Configuration = "127.0.0.1";
		option.InstanceName = "master";
	});
}
{% endhighlight %}

### HomeController 
AddDistributedRedisCache 实际上是将接口IDistributedCache注入 service 集合，因此我们可以在Controller 中通过依赖获取该服务：

{% highlight java %}
[Route("api/[controller]")]
public class HomeController : Controller
{
	private readonly IDistributedCache _distributedCache;
 
	public HomeController(IDistributedCache distributedCache)
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

结果如下：

        Added to cache : 26/06/2017 1:27:24 AM
        *refresh page a few seconds later*
        Fetched from cache : 26/06/2017 1:27:24 AM

### 后记
* IDistributedCache 有异步(async)方法，应该尽可能的使用异步方法
* IDistributedCache 允许保存string 和 byte类型的数据，对于其他类型的，需要序列化为string 或 byte 后保存
* 不仅Redis distributed cache实现了IDistributedCache 接口，其他的例如 InMemory, SQL Server 也实现了此接口，也就是说，Controller中的代码同样可以应用在InMemory Cache 和 SQL Server中
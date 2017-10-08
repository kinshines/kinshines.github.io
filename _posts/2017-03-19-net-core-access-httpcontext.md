---
layout: post
title: .NET Core 获取HttpContext
author: kinshines
date:   2017-03-19
categories: core
permalink: /archivers/net-core-access-httpcontext
---

<p class="lead">HttpContext 在 .NET Core 中的获取方式有些不同，在Controller之外通过 HttpContext.Current 这种方式给单元测试造成较大困难，因此在 .NET Core 已经过时。需要通过以下两种方式获取</p>

### Controllers
{% highlight java %}
[HttpGet]
public async Task<bool> LoggedIn()
{
	var myUser = HttpContext.User;
	return myUser.Identities.Any(x => x.IsAuthenticated);
}
{% endhighlight %}

### Services
首先，在ConfigureServices方法中注册IHttpContextAccessor 作为service：

{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
services.AddMvc();
	services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
}
{% endhighlight %}

然后，在Service中可以通过依赖注入获取：

{% highlight java %}
public class UserService : IUserService
{
	private readonly IHttpContextAccessor _httpContextAccessor;
 
	public UserService(IHttpContextAccessor httpContextAccessor)
	{
		_httpContextAccessor = httpContextAccessor;
	}
 
	public bool IsUserLoggedIn()
	{
		var context = _httpContextAccessor.HttpContext;
		return context.User.Identities.Any(x => x.IsAuthenticated);
	}
}
{% endhighlight %}
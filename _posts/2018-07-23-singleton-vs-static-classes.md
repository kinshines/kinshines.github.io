---
layout: post
title: 单例类(Singleton)对比静态类(Static)
author: kinshines
date:   2018-07-23
categories: core
permalink: /archivers/net-core-access-logging-api-outside-mvc-controller
---

<p class="lead">在.NET Core项目中，微软提供了内置的Logger,通过依赖注入的方式，注入到service层和controller层，使用起来极为方便。但是对于大多数使用过.NET Framework的人来说，更习惯的日志记录方式是通过静态类来实现，那么在静态类中如何使用内置的logger呢？本文将做以下探讨
</p>

我们先创建一个静态帮助类，来成为LoggerFactory的拥有者，如下：

{% highlight java %}

public class ApplicationLogging
{
	private static ILoggerFactory _Factory = null;

	public static void ConfigureLogger(ILoggerFactory factory)
	{
		factory.AddDebug(LogLevel.None).AddStackify();
		factory.AddFile("logFileFromHelper.log"); //serilog file extension
	}

	public static ILoggerFactory LoggerFactory
	{
		get
		{
			if (_Factory == null)
			{
				_Factory = new LoggerFactory();
				ConfigureLogger(_Factory);
			}
			return _Factory;
		}
		set { _Factory = value; }
	}
	public static ILogger CreateLogger() => LoggerFactory.CreateLogger();
}    

{% endhighlight %}


### 怎样捕获ASP.NET 内置的Logging和自己写的Logging
注意：
** ASP.NET只会将内置的Logging写入到app startup处的LoggerFactory **

换言之，如果我们在自己的代码里new一个LoggerFactory()，ASP.NET是不会写入logging的

因此，我们需要在app startup处捕获LoggerFactory：

{% highlight java %}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	//call ConfigureLogger in a centralized place in the code
	ApplicationLogging.ConfigureLogger(loggerFactory);
	//set it as the primary LoggerFactory to use everywhere
	ApplicationLogging.LoggerFactory = loggerFactory;
	//other code removed
}

{% endhighlight %}

通过以上方式，在应用程序中将取得唯一的LoggerFactory，之后我们可以通过ApplicationLogging静态方法取得Logger，希望未来.NET 团队可以简化取得内置logging的方式


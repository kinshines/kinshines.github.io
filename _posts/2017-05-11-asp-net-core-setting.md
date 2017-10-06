---
layout: post
title: ASP.NET Core 获取配置项
author: kinshines
date:   2017-05-11
categories: core
permalink: /archivers/asp-net-core-setting
---

<p class="lead">在 ASP.NET Core 应用中，已无法通过 ConfigurationManager.AppSettings  中获取配置项，另外 ASP.NET Core 也不建议将配置项写在web.config中，而是记在appsettings.json，本文将介绍在 ASP.NET Core 获取配置项的方式</p>


### 定义一个包含配置项的类，Configuration/DemoSettings.cs

        public class DemoSettings
        {
                public string MainDomain { get; set; }
                public string SiteName { get; set; }
        }

appsettings.json 内容如下：

{% highlight js %}
{
  "DemoSettings": {
    "MainDomain": "http://www.mysite.com",
    "SiteName": "My Main Site"
  },
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
{% endhighlight %}

然后，回到Startup.cs文件

### 配置Services

Startup.cs类是定义应用程序所有设置的地方，无论是SQL Server,Mvc,还是Server端服务，其中 ConfigureServices 方法如下：

{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
        // Add framework services.
        services.AddMvc();
}
{% endhighlight %}

需要增加以下配置：

{% highlight java %}
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();

 // Added - uses IOptions<T> for your settings.
    services.AddOptions();

 // Added - Confirms that we have a home for our DemoSettings
    services.Configure<DemoSettings>(Configuration.GetSection("DemoSettings"));
}
{% endhighlight %}

AddOptions()方法是为允许IOptions<T>注入做准备

### 注入(Injecting) 配置项

在Controller中，如果想要获取配置项，需要准备两件事，一是承接配置的属性(property)，二是实现构造注入

Controllers/HomeController.cs

{% highlight java %}
public class HomeController : Controller
{
    private DemoSettings ConfigSettings { get; set; }

    public HomeController(IOptions<DemoSettings> settings)
    {
        ConfigSettings = settings.Value;
    }

 public IActionResult Index()
    {
        ViewData["SiteName"] = ConfigSettings.SiteName;
        return View();
    }
}
{% endhighlight %}
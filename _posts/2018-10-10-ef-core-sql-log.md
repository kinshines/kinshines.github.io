---
layout: post
title: 打印EFCore的数据库操作日志
author: kinshines
date:   2018-10-10
categories: core
permalink: /archivers/ef-core-sql-log
---

<p class="lead">在NET Core程序中，我们常常期望能够直观地查看EF向数据库发送的sql，本文将介绍如何配置以查看sql日志</p>

#### LoggerFactory
首先，我们在Startup类中添加LoggerFactory：

{% highlight java %}
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public static readonly LoggerFactory MyLoggerFactory
    = new LoggerFactory(new[] { new DebugLoggerProvider() });
    …………
    …………
    …………
    }
{% endhighlight %}

#### ConfigureServices
然后，修改Startup类中的ConfigureServices方法，添加上一步新建的MyLoggerFactory
{% highlight java %}
    public void ConfigureServices(IServiceCollection services)
    {
    services.AddDbContext<AppDbContext>(options =>
                options.UseLoggerFactory(MyLoggerFactory).UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    …………
    …………
    …………
    }
{% endhighlight %}

之后，我们再次启动Debug模式，就可以在输出框中看到EF向数据库发送的SQL了

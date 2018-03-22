---
layout: post
title: WebAPI 配置全局异常日志记录
author: kinshines
date:   2018-03-22
categories: webapi
permalink: /archivers/webapi-global-exception-logger
---

<p class="lead">WebAPI 2 中在全局ExceptionFilter之外，增加了异常日志记录器特性，本文将做简单介绍</p>

首先，新建SimpleExceptionLogger类，继承自ExceptionLogger
{% highlight java %}

public class SimpleExceptionLogger : ExceptionLogger
{
private readonly ILog _log;
public SimpleExceptionLogger(ILogManager logManager)
{
_log = logManager.GetLog(typeof (SimpleExceptionLogger));
}
public override void Log(ExceptionLoggerContext context)
{
_log.Error("Unhandled exception", context.Exception);
}
}

{% endhighlight %}

其中ILogManager需要依赖注入，然后再将其作为Service添加至全局配置中，即WebApiConfig类的Register方法：
{% highlight java %}

config.Services.Add(typeof (IExceptionLogger),
new SimpleExceptionLogger(WebContainerManager.Get<ILogManager>()));

{% endhighlight %}

此处的WebContainerManager是项目里面实现的一个简单IOC容器，对于真实项目中，例如使用Autofac时，我们或许无法在Register方法中拿到Container，以上代码可以直接在初始化Autofac的Container后，通过 System.Web.Http.GlobalConfiguration.Configuration 得到config后执行。

此外，在应用程序启动时，尚未执行到Register方法时抛出的异常未能记录下来，所以需要做进一步补充，需要添加HttpApplication类的Error事件：
{% highlight java %}

protected void Application_Error()
{
var exception = Server.GetLastError();
if (exception != null)
{
var log = WebContainerManager.Get<ILogManager>().GetLog(typeof (WebApiApplication));
log.Error("Unhandled exception.", exception);
}
}

{% endhighlight %}

---
layout: post
title: WebAPI 配置全局异常日志记录
author: kinshines
date:   2018-03-22
categories: webapi
permalink: /archivers/webapi-global-exception-logger
---

<p class="lead">WebAPI 2 中增加了异常日志记录特性，本文将做简单介绍</p>

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

此外，为了记录应用程序在启动时抛出的异常，因为此时尚未执行到Register方法，所以需要做进一步补充，需要添加HttpApplication类的Error事件：
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

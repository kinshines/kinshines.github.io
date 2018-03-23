---
layout: post
title: WebAPI 配置全局错误返回信息
author: kinshines
date:   2018-03-23
categories: webapi
permalink: /archivers/webapi-global-error-result
---

<p class="lead">WebAPI 中系统出错后会返回默认的错误信息，暴露了太多详细的错误信息和堆栈信息，在某些场景下不太合适，本文将介绍如何配置自定义的错误返回信息</p>

首先，新建GlobalErrorResult类，继承自IHttpActionResult
{% highlight java %}

public class GlobalErrorResult : IHttpActionResult
{
private readonly string _errorMessage;
private readonly HttpRequestMessage _requestMessage;
private readonly HttpStatusCode _statusCode;
public GlobalErrorResult(HttpRequestMessage requestMessage, HttpStatusCode statusCode,
string errorMessage)
{
_requestMessage = requestMessage;
_statusCode = statusCode;
_errorMessage = errorMessage;
}
public Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
{
return Task.FromResult(_requestMessage.CreateErrorResponse(_statusCode, _errorMessage));
}
}

{% endhighlight %}

然后，再定义GlobalExceptionHandler类，继承自ExceptionHandler：
{% highlight java %}

public class GlobalExceptionHandler : ExceptionHandler
{
public override void Handle(ExceptionHandlerContext context)
{
var exception = context.Exception;
var httpException = exception as HttpException;
if (httpException != null)
{
context.Result = new GlobalErrorResult(context.Request,
(HttpStatusCode) httpException.GetHttpCode(), httpException.Message);
return;
}
context.Result = new GlobalErrorResult(context.Request, HttpStatusCode.InternalServerError,
exception.Message);
}
}

{% endhighlight %}

最后，在WebApiConfig中的Register方法中添加：

{% highlight java %}

config.Services.Replace(typeof (IExceptionHandler), new GlobalExceptionHandler());

{% endhighlight %}

---
layout: post
title: MVC 通过 FileResult 向浏览器发送文件
author: kinshines
date:   2016-08-08
categories: mvc
permalink: /archivers/MVC-FileResult
---

本文转载自[ASP.NET MVC：通过 FileResult 向 浏览器 发送文件](http://www.cnblogs.com/ldp615/archive/2010/09/17/asp-net-mvc-file-result.html)

<p class="lead">在 Controller 中我们可以使用 FileResult 向客户端发送文件</p>

### FileResult
![FileResult](https://kinshines.github.io/img/mvc-fileresult/FileResult_2.png)
FileResult 是一个抽象类，继承自 ActionResult。在 System.Web.Mvc.dll 中，它有如上三个子类，分别以不同的方式向客户端发送文件。

在实际使用中我们通常不需要直接实例化一个 FileResult 的子类，因为 Controller 类已经提供了六个 File 方法来简化我们的操作：
{% highlight java %}
protected internal FilePathResult File(string fileName, string contentType);
protected internal virtual FilePathResult File(string fileName, string contentType, string fileDownloadName);
protected internal FileContentResult File(byte[] fileContents, string contentType);
protected internal virtual FileContentResult File(byte[] fileContents, string contentType, string fileDownloadName);
protected internal FileStreamResult File(Stream fileStream, string contentType);
protected internal virtual FileStreamResult File(Stream fileStream, string contentType, string fileDownloadName);
{% endhighlight %}

### FilePathResult
FilePathResult 直接将磁盘上的文件发送至浏览器：
#### 1. 最简单的方式
{% highlight java %}
public ActionResult FilePathDownload1()
{
    var path = Server.MapPath("~/Files/鹤冲天.zip");
    return File(path, "application/x-zip-compressed");
}
{% endhighlight %}

第一个参数指定文件路径，第二个参数指定文件的 MIME 类型。
用户点击浏览器上的下载链接后，会调出下载窗口：
![FilePathDownload1](https://kinshines.github.io/img/mvc-fileresult/FilePathDownload1_2.png)
大家应该注意到，文件名称会变成 Download1.zip，默认成了 Action 的名字。我们使用 File 方法的第二个重载来解决文件名的问题：
#### 2. 指定 fileDownloadName
{% highlight java %}
public ActionResult FilePathDownload2()
{
    var path = Server.MapPath("~/Files/鹤冲天.zip"); 
    return File("g:\\鹤冲天.zip", "application/x-zip-compressed", "crane.zip");
}

public ActionResult FilePathDownload3()
{
    var path = Server.MapPath("~/Files/鹤冲天.zip"); 
    var name = Path.GetFileName(path);
    return File(path, "application/x-zip-compressed", name);
}
{% endhighlight %}


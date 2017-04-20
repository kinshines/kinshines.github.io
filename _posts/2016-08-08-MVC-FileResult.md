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

## FileResult
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

## FilePathResult
FilePathResult 直接将磁盘上的文件发送至浏览器：
### 1.最简单的方式
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
### 2.指定 fileDownloadName
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
我们可以通过给 fileDownloadName 参数传值来指定文件名，fileDownloadName 不必和磁盘上的文件名一样。下载提示窗口分别如下：
![FilePathDownload2](https://kinshines.github.io/img/mvc-fileresult/FilePathDownload2_2.png)
![FilePathDownload3](https://kinshines.github.io/img/mvc-fileresult/FilePathDownload3_2.png)
FilePathDownload2 没问题，FilePathDownload3 还是默认为了 Action 的名字。原因是 fileDownloadName 将作为 URL 的一部分，只能包含 ASCII 码。我们把 FilePathDownload3 改进一下：
### 3.对 fileDownloadName 进行 Url 编码
{% highlight java %}
public ActionResult FilePathDownload4()
{
    var path = Server.MapPath("~/Files/鹤冲天.zip");
    var name = Path.GetFileName(path);
    return File(path, "application/x-zip-compressed", Url.Encode(name));
}
{% endhighlight %}
再试下，下载窗口如下：
![FilePathDownload4](https://kinshines.github.io/img/mvc-fileresult/FilePathDownload4_2.png)
好了，没问题了。上面代码中 Url.Encode(…)，也可使用 HttpUtility.UrlEncode(…)，前者在内部调用后者。

我们再来看 FileContentResult.
## FileContentResult
FileContentResult 可以直接将 byte[] 以文件形式发送至浏览器（而不用创建临时文件）。参考代码如下：
{% highlight java %}
public ActionResult FileContentDownload1()
{
    byte[] data = Encoding.UTF8.GetBytes("欢迎访问 鹤冲天 的博客 http://www.cnblogs.com/ldp615/");
    return File(data, "text/plain", "welcome.txt");
}
{% endhighlight %}
点击后下载链接后，弹出提示窗口如下：
![FileContentDownload1](https://kinshines.github.io/img/mvc-fileresult/FileContentDownload1_2.png)
## FileStreamResult
想给 FileStreamResult 找一个恰当的例子是不太容易的，毕竟 Http Response 中已经包含了一个输出流，如果要动态生成文件的话，可以直接向这个输出流中写入数据，效率还高。当然，我们不会在 Controller 中直接向 Response 的 OutputStream 写入数据，这样做是不符合MVC的，我们应该把这个操作封装成一个 ActionResult。
不过仔细想想，用途还是有的，比如服务器上有个压缩（或加密）文件，需要解压（或解密）后发送给用户。
### 1. 解压（或解密）
演示代码如下，解压使用 ICSharpCode.SharpZipLib.dll：
{% highlight java %}
public ActionResult FileStreamDownload1()
{
    var path = Server.MapPath("~/Files/鹤冲天.zip");
    var fileStream = new FileStream(path, FileMode.Open);
    var zipInputStream = new ZipInputStream(fileStream);
    var entry = zipInputStream.GetNextEntry();
    return File(zipInputStream, "application/pdf", Url.Encode(entry.Name));
}
{% endhighlight %}
简单起见，假定压缩文件中只有一个文件，且是 pdf 格式的。鹤冲天.zip 如下：
![FileStreamDownload1](https://kinshines.github.io/img/mvc-fileresult/FileStreamDownload1_2.png)
点击后弹出下载提示窗口如下：
![FileStreamDownload1-2](https://kinshines.github.io/img/mvc-fileresult/FileStreamDownload1-2_2.png)
### 2. 转发（或盗链）
FileStreamResult 的另一种用途是将其它网站上的文件作为本站文件下载（其实就是盗链）：
{% highlight java %}
public ActionResult FileStreamDownload2()
{
    var stream = new WebClient().OpenRead("http://files.cnblogs.com/ldp615/Mvc_TextBoxFor.rar");
    return File(stream, "application/x-zip-compressed", "Mvc_TextBoxFor.rar");
}
{% endhighlight %}
看下面提示窗口，来源还是 localhost:
![FileStreamDownload2](https://kinshines.github.io/img/mvc-fileresult/FileStreamDownload2_2.png)
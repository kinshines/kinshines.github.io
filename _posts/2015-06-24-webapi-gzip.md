---
layout: post
title: ASP.NET Web API 使用ActionFilter实现 GZip compression
author: kinshines
date:   2015-06-24
categories: webapi
permalink: /archivers/WebAPI-GZip-Compression
---


<p class="lead">本文演示WebAPI 开启 GZip / Deflate 压缩以提高Web性能</p>
Web压缩是通过压缩传输包大小的方式提高客户端和服务器之前的传输效率。
当前Web传输中有两种通用的成熟压缩算法：GZip 和 Deflate，这两种算法都能被浏览器识别而且在HTTP的响应中可以自动解压缩。
下图展示了GZip压缩效果：
![Compression](http://blog.developers.ba/wp-content/uploads/2014/06/compression.png "Compression")
图片来源：<a href="http://www.sendung.de/2007-04-09/web-services-output-formats-and-gzip-compression/">Effects of GZip compression</a>

目前在ASP.NET Web API上实现压缩有以下三种方式：

1.IIS级别，在IIS中设置开启即可

2.自定义的委托Handler级别

3.自定义的ActionFilter，可应用在Method级别，Controller级别甚至整个WebAPI级别

<p class="lead">下面演示ActionFilter实现GZip压缩</p>
### DotNetZip
首先要借助第三方库<a href="http://dotnetzip.codeplex.com/">DotNetZip library</a>
Nuget获取：

{% highlight js %}

Install-Package DotNetZip

{% endhighlight %}

### ActionFilter
下面实现Deflate 压缩ActionFilter
{% highlight java %}

public class DeflateCompressionAttribute : ActionFilterAttribute
{
 
   public override void OnActionExecuted(HttpActionExecutedContext actContext)
   {
       var content = actContext.Response.Content;
       var bytes = content == null ? null : content.ReadAsByteArrayAsync().Result;
       var zlibbedContent = bytes == null ? new byte[0] : 
       CompressionHelper.DeflateByte(bytes);
       actContext.Response.Content = new ByteArrayContent(zlibbedContent);
       actContext.Response.Content.Headers.Remove("Content-Type");
       actContext.Response.Content.Headers.Add("Content-encoding", "deflate");
       actContext.Response.Content.Headers.Add("Content-Type","application/json");
       base.OnActionExecuted(actContext);
     }
 }

{% endhighlight %}

上文中的CompressionHelper定义如下：
{% highlight java %}

public class CompressionHelper
{ 
        public static byte[] DeflateByte(byte[] str)
        {
            if (str == null)
            {
                return null;
            }
 
            using (var output = new MemoryStream())
            {
                using (
                    var compressor = new Ionic.Zlib.DeflateStream(
                    output, Ionic.Zlib.CompressionMode.Compress, 
                    Ionic.Zlib.CompressionLevel.BestSpeed))
                {
                    compressor.Write(str, 0, str.Length);
                }
 
                return output.ToArray();
            }
        }
}

{% endhighlight %}

对于GZipCompressionAttribute的实现和DeflateCompressionAttribute基本一致，需要改动的地方只是将上文中的
DeflateStream改为GZipStream

### 应用：
在方法上添加属性标识实现：

{% highlight java %}

public class V1Controller : ApiController
{
   
    [DeflateCompression]
    public HttpResponseMessage GetCustomers()
    {
 
    }
 
}

{% endhighlight %}
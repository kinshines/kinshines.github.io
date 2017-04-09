---
layout: post
title: ASP.NET Web API 使用Jil Json作为默认json序列化工具
author: kinshines
date:   2015-06-22
categories: webapi
permalink: /archivers/WebAPI-JilJson
---

Web API 默认的json序列化工具是JavaScriptSerializer，这个工具性能不太好，微软都不推荐使用，
网上有人当今流行的的json序列化工具做过性能对比
![json serialize time](http://blog.developers.ba/wp-content/uploads/2014/07/JilSpeed_thumb.png)
本文将使用Jil Json 替换Web API 默认的json序列化工具

### Nuget 安装 jil
{% highlight js %}

Install-Package Jil

{% endhighlight %}

### 创建 Jil MediaTypeFormatter
{% highlight java %}

public class JilFormatter : MediaTypeFormatter
{
    private readonly Options _jilOptions;
    public JilFormatter()
    {
        _jilOptions=new Options(dateFormat:DateTimeFormat.ISO8601);
        SupportedMediaTypes.Add(new MediaTypeHeaderValue("application/json"));
 
        SupportedEncodings.Add(new UTF8Encoding(encoderShouldEmitUTF8Identifier: false, throwOnInvalidBytes: true));
        SupportedEncodings.Add(new UnicodeEncoding(bigEndian: false, byteOrderMark: true, throwOnInvalidBytes: true));
    }
    public override bool CanReadType(Type type)
    {
        if (type == null)
        {
            throw new ArgumentNullException("type");
        }
        return true;
    }
 
    public override bool CanWriteType(Type type)
    {
        if (type == null)
        {
            throw new ArgumentNullException("type");
        }
        return true;
    }
 
    public override Task<object> ReadFromStreamAsync(Type type, Stream readStream, System.Net.Http.HttpContent content, IFormatterLogger formatterLogger)
    {
        return Task.FromResult(this.DeserializeFromStream(type, readStream));           
    }
 
 
    private object DeserializeFromStream(Type type,Stream readStream)
    {
        try
        {
            using (var reader = new StreamReader(readStream))
            {
                MethodInfo method = typeof(JSON).GetMethod("Deserialize", new Type[] { typeof(TextReader),typeof(Options) });
                MethodInfo generic = method.MakeGenericMethod(type);
                return generic.Invoke(this, new object[]{reader, _jilOptions});
            }
        }
        catch
        {
            return null;
        }
 
    }
 
 
    public override Task WriteToStreamAsync(Type type, object value, Stream writeStream, System.Net.Http.HttpContent content, TransportContext transportContext)
    {
        var streamWriter = new StreamWriter(writeStream);
        JSON.Serialize(value, streamWriter, _jilOptions);
        streamWriter.Flush();
        return Task.FromResult(writeStream);
    }
}

{% endhighlight %}

### 替换默认的 JSON serializer
在WebApiConfig类的Register方法开头处添加如下代码：

{% highlight java %}

config.Formatters.RemoveAt(0);
config.Formatters.Insert(0, new JilFormatter());

{% endhighlight %}

### 使用Json.Net
C#里的json工具最常用的是Json.Net，当然也可以用Json.Net作为默认的序列化工具，原理同上。
利用Nuget已有的资源
{% highlight java %}

Install-Package JsonNetMediaTypeFormatter

{% endhighlight %}

{% highlight java %}

config.Formatters.RemoveAt(0);
config.Formatters.Insert(0, new JsonNetMediaTypeFormatter());

{% endhighlight %}
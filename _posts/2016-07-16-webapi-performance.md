---
layout: post
title: 8 种提高ASP.NET Web API 性能的方法
author: kinshines
date:   2016-07-16
categories: webapi
permalink: /archivers/WebAPI-improve-performance
---

<p class="lead"> ASP.NET Web API 是实现Web api极为卓越的工具，但是我们多好开发者没有深入了解如何提高其性能。
本文将探寻8种提高Web API 性能的方法</p>

### 1.使用更快的json序列化工具
ASP.NET Web API默认采用的是JavaScriptSerializer，性能不佳，可以替换成效率更高的json序列化工具
下图展示了较为流行的json序列化工具的比较：
![json serializer performance](http://blog.developers.ba/wp-content/uploads/2014/07/SerializerPerformanceGraf_thumb.png)
图片来源：[theburningmonk](http://theburningmonk.com/2014/06/json-and-binary-serializers-benchmarks-updated/)
StackOverflow 声称他们使用了更快的序列化工具 [Jil serializer](https://github.com/kevin-montrose/Jil)
我的另一篇博客详细介绍了如何替换json序列化工具 [ASP.NET Web API 使用Jil Json作为默认json序列化工具](https://kinshines.github.io/archivers/WebAPI-JilJson)

### 2.从DataReader里直接json序列化
WebAPI返回值通常是先从DateReader里遍历原始值再实例化成类的对象，继而将该对象json序列化，
所以我们考虑直接从DateReader里手动json序列化而免去创建实例的过程，具体做法是使用StringBuilder构建StringContent
作为WebAPI的响应
{% highlight java %}

var response = Request.CreateResponse(HttpStatusCode.OK);
response.Content = new StringContent(jsonResult, Encoding.UTF8, "application/json");
return response;

{% endhighlight %}

将DateReader实现json序列化：
{% highlight java %}
string jsonResult;
var serializer = new WestwindJsonSerializer
{
     DateSerializationMode = JsonDateEncodingModes.Iso
};
 
using (SqlDataReader reader = cmd.ExecuteReader())
{ 
     jsonResult = serializer.Serialize(reader);
}
{% endhighlight %}

### 3.尝试使用除json之外的数据格式（protocol buffer, message pack）
条件允许的话，可以尝试使用Protocol Buffers 或者 MessagePack 来传输数据，Protocol Buffers不仅序列化更快，而且相比于json数据格式更小

### 4.实现压缩
在WebAPI中实现GZIP 或 Deflate 压缩
压缩可以使传输量更小进而效率更高，具体可以参考[Web API 实现GZIP和Deflate压缩](https://kinshines.github.io/archivers/WebAPI-GZip-Compression)

### 5.缓存
缓存是提高性能的屡试不爽的方法，下面这段简单代码是基于微软的System.Runtime.Caching dll，当然也可以使用redis,memcache等缓存策略
{% highlight java %}
public class MemoryCacher
{
  public object GetValue(string key)
  {
    MemoryCache memoryCache = MemoryCache.Default;
    return memoryCache.Get(key);
  }
 
  public bool Add(string key, object value, DateTimeOffset absExpiration)
  {
    MemoryCache memoryCache = MemoryCache.Default;
    return memoryCache.Add(key, value, absExpiration);
  }
 
  public void Delete(string key)
  {
    MemoryCache memoryCache = MemoryCache.Default;
    if (memoryCache.Contains(key))
    {
       memoryCache.Remove(key);
    }
  }
}
{% endhighlight %}

用法：
{% highlight java %}
memCacher.Add(token, userId, DateTimeOffset.UtcNow.AddHours(1));

var result = memCacher.GetValue(token);
{% endhighlight %}

### 6.尽可能是使用经典的ADO.NET
硬编码的ADO.NET仍然是从数据库读取数据最快的方式，对于性能要求苛刻的代码，还是不要使用ORM了
下图展示了流行的ORM之间的性能比较：
![ORMMapper](http://blog.developers.ba/wp-content/uploads/2014/07/ORMMapper_thumb.png)

### 7.Web API使用关键字async实现异步方法
使用异步Web API服务可以提高Web API能够处理的HTTP请求的并发量。
实现起来也很容易，方法前面使用async关键字标记并且返回类型改为Task
{% highlight java %}
[HttpGet]  
public async Task OperationAsync()  
{   
    await Task.Delay(2000);  
}
{% endhighlight %}

### 8.返回多数据集
查询数据库时，可以将多个数据集合并返回
{% highlight java %}
[HttpGet]  
// read the first resultset 
var reader = command.ExecuteReader(); 
 
// read the data from that resultset 
while (reader.Read()) 
{ 
    suppliers.Add(PopulateSupplierFromIDataReader( reader )); 
} 
 
// read the next resultset 
reader.NextResult(); 
 
// read the data from that second resultset 
while (reader.Read()) 
{ 
    products.Add(PopulateProductFromIDataReader( reader )); 
}
{% endhighlight %}

Web API 的响应中也可以组合返回
{% highlight java %}
public class AggregateResult
{
     public long MaxId { get; set; }
     public List<Folder> Folders{ get; set; }
     public List<User>  Users{ get; set; }
}
{% endhighlight %}

这样可以减少HTTP请求次数。
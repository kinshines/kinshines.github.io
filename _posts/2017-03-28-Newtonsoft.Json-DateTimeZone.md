---
layout: post
title: Newtonsoft.Json 时区差解决方法
author: kinshines
date:   2017-03-28
categories: C#
permalink: /archivers/Newtonsoft-Json-DateTimeZone
---

今天和小马哥对接一个api接口，他返回给我的一个json里包含了一个DateTime类型的值，
我接收并使用Newtonsoft.Json的JsonConvert.DeserializeObject<T>()方法反序列化后得到的该DateTime值
始终比他序列化之前的值要小8个小时，经过排查，原因是他使用的json序列化工具是JavaScriptSerializer.Serialize()，
它在序列化DateTime类型的值的时候，会先将该时间转化成UTC时间，因此我使用Newtonsoft.Json反序列化时需要如下设置：
{% highlight js %}
JsonConvert.DeserializeObject<T>(jsonStr, new JsonSerializerSettings
            {
                DateTimeZoneHandling = DateTimeZoneHandling.Local
            });
{% endhighlight %}

后记：JavaScriptSerializer是.net Framework里自带的类库，性能相比于Newtonsoft.Json较差，微软也不推荐使用。

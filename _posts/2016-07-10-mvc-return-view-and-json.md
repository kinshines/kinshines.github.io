---
layout: post
title: MVC action中同时返回View和Json
author: kinshines
date:   2016-07-10
categories: mvc
permalink: /archivers/mvc-return-view-and-json
---

<p class="lead"> 今天遇到一个需求，希望在action在返回普通View视图的同时，一并返回model的Json，解决方式如下，以此记录</p>

首先，action在return View()之前，定义一个ViewBag.Json来接受json序列化的model
{% highlight java %}
public ActionResult ExampleAction()
{
    ...
    ViewBag.Json=JsonConvert.SerializeObject(model);
    return View();
}
{% endhighlight %}

在前台View中，如果直接用js中来承接ViewBag.Json，得到的是经过html编码后的json，例如将双引号"编码成了&quot;显然无法将其作为一个json来使用了，这里我的解决方案是定义一个隐藏的div标签来承接ViewBag.Json，然后通过获取该div的text来取得原始的json
{% highlight html %}
<div id='json-data-holder' class='hidden'>ViewBag.Json</div>
<script>
var json=JSON.parse($('#json-data-holder').text());
</script>
{% endhighlight %}


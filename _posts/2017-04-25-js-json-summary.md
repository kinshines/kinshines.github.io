---
layout: post
title: javascript中判断json的方法总结
author: kinshines
date:   2017-04-25
categories: js
permalink: /archivers/js-json-summary
---

<p class="lead"> JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式，采用完全独立于语言的文本格式，是理想的数据交换格式。同时，JSON是 JavaScript 原生格式，这意味着在 JavaScript 中处理 JSON数据不须要任何特殊的API或工具包，本文将总结js中判断json的方法。</p>

### 1.json定义：
{% highlight js %}
var jsonStr={}
{% endhighlight %}

### 2.判断json是否为空：
{% highlight js %}
jQuery.isEmptyObject()
{% endhighlight %}

### 3.判断对象是否为空：
{% highlight js %}
3.1 if(typeof(x)   ==   "undefined")

3.2 if(typeof(x)   !=   "object")

3.3 if(!x)
{% endhighlight %}
其中第三种是最简单的方法，但是第三种就不能用if(x)这种互斥的方法去判断，只能在对象前面加!

### 4.json的key是不可以重复的：
jsonStr[key]="xxx",存在在替换，不存在则新增

### 5.遍历json
{% highlight js %}
for(var key in jsonStr){
   alert(key+" "+jsonStr[key])
}
{% endhighlight %}

### 6.判断返回是否是json格式：
{% highlight js %}
isJson = function(obj){
    var isjson = typeof(obj) == "object" && Object.prototype.toString.call(obj).toLowerCase() == "[object object]" && !obj.length;
    return isjson;
}
if (!isJson(data)) 
    data = eval('('+data+')');//将字符串转换成json格式
{% endhighlight %}


---
layout: post
title: js中contains方法和indexOf方法
author: kinshines
date:   2016-05-05
categories: js
permalink: /archivers/js-contains-indexof
---

在后端语言中，contains可以用于判断str1是否包含str2

原生js中是有contains方法的，但它并不是字符串方法，而仅用于判断DOM元素的包含关系，参数是Element类型
{% highlight js %}
var div1=document.getElementById("div1");
var div2=document.getElementById("div2");
alert(div1.contains(div2))//true
{% endhighlight %}

如果字符串之间的判断使用此方法则报错
{% highlight js %}
var str1="1234";
var str2="12";
alert(str1.contains(str2))//error
{% endhighlight %}

若要在js中判断俩字符串的包含关系，需要用indexOf()
{% highlight js %}
var str1="1234";
var str2="12";
alert(str1.indexOf(str2))//0
{% endhighlight %}

一个小误区，希望以后注意

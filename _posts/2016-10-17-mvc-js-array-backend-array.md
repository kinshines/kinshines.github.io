---
layout: post
title: MVC 前台js发送数组，后台用数组接收
author: kinshines
date:   2016-10-14
categories: mvc
permalink: /archivers/mvc-js-array-backend-array
---

<p class="lead">今天遇到一个问题，在js中使用ajax提交一个json数组，希望Controller中action的可以直接接收到此数组，ajax写法与普通的写法有少许不同，记录如下</p>

前台js：
{% highlight js %}
//定义一个json数组
var idsArr=new Array();
idsArr.push(1);
idsArr.push(2);
idsArr.push(3);

//ajax方法
$.ajax({
url: "/Home/UpdateByIds",
type: "post",
traditional: true,//布尔值，规定是否使用参数序列化的传统样式，这样后台数组变量才能接受到值
data: {ids: idsArr},
dataType: "json",
success: function (data) {
}
});
{% endhighlight js %}
需要注意的是 traditional: true

action设置参数：int[] ids
{% highlight java %}
public ActionResult UpdateByIds(int[] ids)  
{}  
{% endhighlight %}

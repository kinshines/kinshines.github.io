---
layout: post
title: 解决Microsoft.jQuery.Unobtrusive.Validation在IE浏览器下校验DateTime失败的问题
author: kinshines
date:   2017-04-17
categories: js
permalink: /archivers/Unobtrusive-Validation-IE-DateTime
---

### 问题回顾
今天遇到一个关于[Microsoft.jQuery.Unobtrusive.Validation](https://www.nuget.org/packages/Microsoft.jQuery.Unobtrusive.Validation/)的问题，
情况如下：在IE浏览器下，文本框输入时间格式如2017-04-17 13:23:54的话，
提交表单时，校验date格式失败，无法提交，而在Chrome和Firefox以及Edge下则没有问题，
真心为微软捏一把汗，自家出的js插件竟然没有做好自家浏览器的兼容。
### 解决思路
输入时间格式时，我们往往使用js时间选择器[Bootstrap 3 Datetimepicker](https://www.nuget.org/packages/Bootstrap.v3.Datetimepicker/)来选择输入，而非手动填写。
此插件完全可以控制时间格式的有效性，因此可以去除Microsoft.jQuery.Unobtrusive.Validation对于date格式的校验。
### 解决方案
修改jquery.validate.unobtrusive.js源码，大概在372行的位置，移除：
{% highlight js %}
.addBool("date")
{% endhighlight %}

修改jquery.validate.unobtrusive.min.js源码，移除：
{% highlight js %}
.addBool("date")
{% endhighlight %}


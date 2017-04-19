---
layout: post
title: MVC 关于防止跨域脚本攻击AntiXSS
author: kinshines
date:   2016-06-09
categories: mvc
permalink: /archivers/MVC-AntiXSS-Package
---

使用MVC提供文本编辑器，保存用户输入的文本时，需要过滤掉危险字符，微软提供了一个有效的工具AntiXSS
nuget安装包：
{% highlight js %}

Install-Package AntiXSS

{% endhighlight %}

ViewModel中相关属性：

{% highlight java %}

[DisplayName("內容")]
[UIHint("Html")]
[AllowHtml]
public string ContentText { get; set; }

{% endhighlight %}

调用方法：

{% highlight java %}

Microsoft.Security.Application.Sanitizer.GetSafeHtmlFragment(ContentText);

{% endhighlight %}

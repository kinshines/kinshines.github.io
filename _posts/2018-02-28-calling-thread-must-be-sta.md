---
layout: post
title: 在C# 代码中访问Resource
author: kinshines
date:   2018-02-28
categories: wpf
permalink: /archivers/calling-thread-must-be-sta
---

<p class="lead">在WPF项目中，在线程中操作UI，报出以下错误：<br/>
The calling thread must be STA, because many UI components require this. <br/>
本文记录解决方案</p>

### MvvmLightLibs

首先想到借助MvvmLightLibs类库的DispatcherHelper.RunAsync(Action action)方法，
虽然不再报错了，但是UI界面也并未更新，于是进一步探索。

### Application.Current.Dispatcher.Invoke

又尝试解决方案如下：
{% highlight java %}

Application.Current.Dispatcher.Invoke((Action)delegate{

//your code

});

{% endhighlight %}

搞定


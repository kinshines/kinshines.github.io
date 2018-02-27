---
layout: post
title: 在C# 代码中访问Resource
author: kinshines
date:   2018-02-27
categories: wpf
permalink: /archivers/cs-using-resource
---

<p class="lead">在WPF中通常在xmal前台界面中使用DynamicResource和StaticResource资源，本文将介绍如何在cs后台代码中访问Resource资源</p>

### 定义资源

{% highlight java %}

Window.Resources.Add(“backgroundBrush”, new SolidColorBrush(“Yellow”));
Window.Resources.Add(“borderBrush”, new SolidColorBrush(“Red”));

{% endhighlight %}

### StaticResource

{% highlight java %}

Button button = new Button();
button.Background = (Brush)button.FindResource(“backgroundBrush”);
button.BorderBrush = (Brush)button.FindResource(“borderBrush”);

{% endhighlight %}

FindResource没找到会异常，或者用TryFindResource方法，没找到返回null

### DynamicResource

{% highlight java %}

Button button = new Button();
button.SetResourceReference(Button.BackgroundProperty, “backgroundBrush”);
button.SetResourceReference(Button.BorderBrushProperty, “borderBrush”);

{% endhighlight %}

从这里可以看到DynamicResource只能在Dependency property上使用。

虽然可以直接使用索引器检索到资源：

{% highlight java %}

Button button = new Button();
button.Background = (Brush)window.Resources[“backgroundBrush”];
button.BorderBrush = (Brush)window.Resources[“borderBrush”];

{% endhighlight %}

但是这种方法是不提倡的，因为直接检索Resource dictionary，不遍历逻辑树，某些时候会产生非预期效果，当然不遍历逻辑树，性能上有一点点提升。
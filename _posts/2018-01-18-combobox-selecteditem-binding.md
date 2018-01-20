---
layout: post
title: ComboBox
author: kinshines
date:   2018-01-18
categories: wpf
permalink: /archivers/combobox-selecteditem-binding
---

<p class="lead">在WPF中，使用控件ComboBox Binding方式绑定SelectedItem时，一直绑定不上，下面提供一种解决方案</p>

### 需求
比如要做一个打印机列表，从中选择一个打印机（System.Printing），但是以下代码绑定无效：
{% highlight xml %}
            <ComboBox Width="150" 
                      ItemsSource="{Binding PrintQueues}" 
                      SelectedItem="{Binding Model.CurrentPrintQueue}" 
                      DisplayMemberPath="Name">
            </ComboBox>
{% endhighlight %}

{% highlight java %}
            var printServer = new LocalPrintServer();
            PrintQueues = printServer.GetPrintQueues();
            Model.CurrentPrintQueue = printServer.DefaultPrintQueue;
{% endhighlight %}

### 解决方案
首先想SelectedItem肯定是来自于ItemsSource中的一个引用，并且他们是同一个对象才行。
但是看代码，我以为printServer.DefaultPrintQueue就是GetPrintQueues()中的一个对象，所以代码后台修改为：

{% highlight java %}
Model.CurrentPrintQueue = PrintQueues.FirstOrDefault(x => x.Name == printServer.DefaultPrintQueue.Name);
{% endhighlight %}

所以谨记，SelectedItem必须是ItemsSource中的同一个对象
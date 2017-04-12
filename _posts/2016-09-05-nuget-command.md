---
layout: post
title: NuGet常用命令
author: kinshines
date:   2016-09-05
categories: Developer
permalink: /archivers/NuGet-Command
---

## NuGet常用命令

### 1. 安装最新版本
{% highlight js %}
install-package unity
{% endhighlight %}

### 2. 安装指定版本
{% highlight js %}
install-package unity -version 3.5.14
{% endhighlight %}

### 3. 安装到指定的项目
{% highlight js %}
install-package unity -project XXXProjectName -version 3.5.14
{% endhighlight %}

### 4.重新安装所有Nuget包(整个解决方案都会重新安装)
{% highlight js %}
update-package -reinstall
{% endhighlight %}
### 5.重新安装指定项目所有Nuget包
{% highlight js %}
update-package -project XXXProjectName -reinstall
{% endhighlight %}

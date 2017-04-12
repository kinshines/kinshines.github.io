---
layout: post
title: NuGet常用命令
author: kinshines
date:   2016-09-05
categories: Developer
permalink: /archivers/NuGet-Command
---

### NuGet常用命令
1. 安装最新版本
{% highlight js %}

install-package unity

{% endhighlight %}
2. 安装指定版本
{% highlight js %}

install-package unity -version 3.5.14

{% endhighlight %}
### 更改nuget资源位置
<p class="lead">在VS中，nuget包资源会默认下载到各项目的pakeages文件夹下，如果项目很多，不便于对资源进行集中管理，可在解决方案同级目录下添加 nuget.config 文件，实现所有资源全部放在上级lib文件夹中</p>
{% highlight js %}

<settings>  
<repositoryPath>..\lib</repositoryPath>  
</settings>  

{% endhighlight %}
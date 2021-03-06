---
layout: post
title: NuGet常用命令
author: kinshines
date:   2015-12-15
categories: develop
permalink: /archivers/NuGet-Command
---

### NuGet常用命令

#### 1. 安装最新版本
{% highlight js %}
install-package unity
{% endhighlight %}
#### 2. 安装指定版本
{% highlight js %}
install-package unity -version 3.5.14
{% endhighlight %}
#### 3. 安装到指定的项目
{% highlight js %}
install-package unity -project XXXProjectName -version 3.5.14
{% endhighlight %}
#### 4. 重新安装所有Nuget包(整个解决方案都会重新安装)
{% highlight js %}
update-package -reinstall
{% endhighlight %}
#### 5. 重新安装指定项目所有Nuget包
{% highlight js %}
update-package -project XXXProjectName -reinstall
{% endhighlight %}
#### 6. 重新安装单个Nuget包
{% highlight js %}
update-package unity -reinstall
{% endhighlight %}
#### 7. 卸载Nuget包
{% highlight js %}
uninstall-package unity
{% endhighlight %}
#### 8. 更新Nuget包至最新版
{% highlight js %}
update-package unity
{% endhighlight %}
#### 9. 更新Nuget包至指定版本
{% highlight js %}
update-package unity -version 4.0.1
{% endhighlight %}
#### 10. 默认列出本地已经安装了的包
{% highlight js %}
get-package
{% endhighlight %}
可以加参数 -filter EntityFramework 筛选自己想要的包

### 更改nuget资源位置
<p class="lead">在VS中，nuget包资源会默认下载到各项目的packages文件夹下，如果项目很多，不便于对资源进行集中管理，可在解决方案同级目录下添加 nuget.config 文件，实现所有资源全部放在上级lib文件夹中</p>

{% highlight js %}
<settings>  
<repositoryPath>..\lib</repositoryPath>  
</settings>  
{% endhighlight %}

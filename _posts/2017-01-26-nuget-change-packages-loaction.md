---
layout: post
title: 修改Nuget下载的package文件夹位置
author: kinshines
date:   2017-01-26
categories: develop
permalink: /archivers/nuget-change-packages-location
---

<p class="lead">Nuget下载package时，默认会在解决方案(.sln)同级目录下，创建packages文件夹以保存下载下来的package，这样可以使得同一解决方案下的各项目共用package，而我们如果还想再进一步，使不同的解决方案之间共用package，那就需要在解决方案(.sln)同级目录下创建nuget.config文件指定package目录</p>

### Nuget 2.1 之前（VS2012，VS2013）
nuget.config内容如下：
{% highlight xml %}
<settings>
<repositoryPath>{some path here}</repositoryPath>
</settings>
{% endhighlight %}

### Nuget 2.1 及之后（VS2015，VS2017）
nuget.config内容如下：
{% highlight xml %}
<configuration>
  <config>
    <add key="repositoryPath" value="{some path here}" />
  </config>
</configuration>
{% endhighlight %}

其中repositoryPath的value值即可以是绝对目录如：C:\thePathToMyPackagesFolder，也可以是相对目录如：../packages


延伸阅读：

[package folder override](https://www.nuget.org/packages/NuGetPackageFolderOverride)

[release notes for 2.1](https://docs.microsoft.com/en-us/nuget/release-notes/nuget-2.1#Specify_%e2%80%98packages%e2%80%99_Folder_Location)



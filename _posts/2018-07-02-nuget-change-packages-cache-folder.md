---
layout: post
title: 修改Nuget下载的package缓存文件夹
author: kinshines
date:   2018-07-02
categories: develop
permalink: /archivers/nuget-change-packages-cache-folder
---

<p class="lead">.NET Core项目中nuget的package不再添加到解决方案目录下，而是统一存放在缓存文件夹下，导致nuget缓存文件夹非常大，默认的nuget缓存文件夹是在C盘，所以需要移动它。
</p>

可以使用下面的代码查看 nuget 全局缓存文件所在的文件夹

        nuget locals all -list

可以看到下面的输出

        http-cache: C:\Users\user\AppData\Local\NuGet\v3-cache   #NuGet 3.x+ cache
        packages-cache: C:\Users\user\AppData\Local\NuGet\Cache  #NuGet 2.x cache
        global-packages: C:\Users\user\.nuget\packages\          #Global packages folder
        temp: C:\Users\user\AppData\Local\Temp\NuGetScratch      #Temp folder

这样可以看到，所在的全局缓存文件夹是放在 C 盘，那么我提供两个方法可以修改：

### 修改链接
可以使用管理员权限运行 PowerShell 来进行文件夹链接，首先复制 nuget 的 package 文件夹到 另外的地方，我移动到D:\nuget\packages，所以就可以使用下面代码把 nuget 文件夹移动到另一个文件夹

        mklink /d C:\Users\user\.nuget\packages D:\nuget\packages

在使用这个代码之前，需要删除 C:\Users\user\.nuget\packages 请把这个字符串修改为自己的 nuget 文件夹

### 配置
除了上面的方法，还可以通过修改配置，修改全局文件夹

打开 %AppData%\NuGet\NuGet.Config ，在这个文件夹添加下面代码：

{% highlight xml %}
<configuration>
  <config>
     <add key="globalPackagesFolder" value="D:\nuget\packages" />
  </config>
</configuration>
{% endhighlight %}

### NuGet Cache
#### Mac
* ~/.local/share/NuGet/Cache
* ~/.nuget/packages
#### Windows
* %LocalAppData%\NuGet\Cache
* %UserProfile%\.nuget\packages
#### Linux
* ~/.local/share/NuGet/Cache 
* ~/.nuget/packages

### NuGet Configuration
#### Mac 
~/.config/NuGet/NuGet.Config
#### Windows 
%AppData%\NuGet\NuGet.Config
#### Linux 
~/.config/NuGet/NuGet.Config

延伸阅读：

[NuGet File Locations - Matt Ward](http://lastexitcode.com/projects/NuGet/FileLocations/)
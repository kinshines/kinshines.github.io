---
layout: post
title: MVC 5 在Windows Server 2008/IIS 7的部署HTTP Error 403.14问题
author: kinshines
date:   2015-09-18
categories: mvc
permalink: /archivers/MVC5-IIS7-HTTP-Error-403
---

MVC 5 应用部署到Windows Server 2008/IIS 7后，运行可能会遇到以下问题：
HTTP Error 403.14 - Forbidden The Web server is configured to not list the contents of this directory.
网上有种解决方案是web.config中配置：

{% highlight xml %}
<modules runAllManagedModulesForAllRequests="true">
{% endhighlight %}

这种方案虽然可以运行正常，但是明显加大了IIS负荷，因为一些静态文件css,js等不需要走ManagedModule，
所以此方案不可采用
正确的解决方案应该是安装IIS更新
[更新是可使某些 IIS 7.0 或 IIS 7.5 处理程序来处理请求的 Url 不以句号结尾](https://support.microsoft.com/zh-cn/help/980368/a-update-is-available-that-enables-certain-iis-7.0-or-iis-7.5-handlers-to-handle-requests-whose-urls-do-not-end-with-a-period)

之后在web.config中配置：
{% highlight xml %}

<configuration>
  <system.webServer>
    <handlers>
      <remove name="ExtensionlessUrl-Integrated-4.0" />
      <remove name="ExtensionlessUrl-ISAPI-4.0_64bit" />
      <remove name="ExtensionlessUrl-ISAPI-4.0_32bit" />
    </handlers>
  </system.webServer>
</configuration>

{% endhighlight %}

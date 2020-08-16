---
layout: post
title: 隐藏IIS默认响应头
author: kinshines
date:   2020-07-29
categories: mvc
permalink: /archivers/response-hide-iis-version
---

<p class="lead">IIS安全基线要求response中不能泄露版本信息，下文介绍如何配置</p>
在IIS+ASP.NET的运行环境，默认情况下会输出以下的响应头（Response Headers）：

        Server    Microsoft-IIS/8.0
        X-AspNet-Version    4.0.30319
        X-AspNetMvc-Version    5.1
        X-Powered-By    ASP.NET

### 移除Sever
1.下载重写模块
[URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)

2.修改全局配置文件 C:\Windows\System32\inetsrv\config\applicationHost.config
在<system.webServer>中添加以下配置
{% highlight xml %}
    <rewrite>
        <allowedServerVariables>
            <add name="REMOTE_ADDR" />
        </allowedServerVariables>            
        <outboundRules>
            <rule name="REMOVE_RESPONSE_SERVER">
                <match serverVariable="RESPONSE_SERVER" pattern=".*" />
                <action type="Rewrite" />
            </rule>
        </outboundRules>
    </rewrite>
{% endhighlight %}

### 移除X-AspNet-Version

在web.config的<httpRuntime>中添加enableVersionHeader="false"：
{% highlight xml %}
<httpRuntime enableVersionHeader="false" />
{% endhighlight %}

### 移除X-AspNetMvc-Version

在 Application_Start() 中添加如下代码：

{% highlight java %}
protected void Application_Start()
{
    MvcHandler.DisableMvcResponseHeader = true;
}
{% endhighlight %}

### 移除X-Powered-By
在IIS Manager功能视图中，IIS=>HTTP 响应标头，移除X-Powered-By

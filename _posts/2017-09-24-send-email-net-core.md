---
layout: post
title: .NET Core 2.0 发送 Email
author: kinshines
date:   2017-09-24
categories: core
permalink: /archivers/sending-email-net-core-2
---

<p class="lead">由于在.NET Core 1.0 版本中，SMTP client没有实现，导致无法发送邮件，随着.NET Core 2.0版本的发布，这个问题顺利解决，喜大普奔</p>

{% highlight java %}
SmtpClient client = new SmtpClient("mysmtpserver");
client.UseDefaultCredentials = false;
client.Credentials = new NetworkCredential("username", "password");
 
MailMessage mailMessage = new MailMessage();
mailMessage.From = new MailAddress("whoever@me.com");
mailMessage.To.Add("receiver@me.com");
mailMessage.Body = "body";
mailMessage.Subject = "subject";
client.Send(mailMessage);
{% endhighlight %}

以上代码和full framework 里的实现基本没有差别，所以也就是复制粘贴的事，不过要注意的是，要保证已升级至 .NET Core 2.0 框架，检查csproj文件：

{% highlight xml %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>
</Project>
{% endhighlight %}

在这里TargetFramework要设置为netcoreapp2.0
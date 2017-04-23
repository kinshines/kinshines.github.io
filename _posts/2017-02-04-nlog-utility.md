---
layout: post
title: NLog使用帮助
author: kinshines
date:   2017-02-04
categories: c#
permalink: /archivers/nlog-utility
---

<p class="lead">本文是我对NLog的使用心得</p>

[NLog](http://nlog-project.org/)是.NET平台下一款优秀的日志工具，它提供友好完善的日志方法，丰富的输出配置，下面是我个人对NLog使用总结。

完整代码参见[Github-NLogUtility](https://github.com/kinshines/NLogUtility/tree/master/NLogUtility)

Nuget源[Nuget-NLogUtility](https://www.nuget.org/packages/NLogUtility)

### 日志记录部分
{% highlight java %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace NLogUtility
{
    public class Logger
    {
        public static void Info(string module, string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger(module);
            logger.Info(message, args);
        }
        public static void PureInfo(string module, string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger(module + ".pure");
            logger.Info(message, args);
        }

        public static void Trace(string module, string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger(module);
            logger.Trace(message, args);
        }

        public static void Error(Exception ex, string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger("error");
            logger.Error(ex, message, args);
        }
        public static void Error(Exception ex)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger("error");
            logger.Error(ex);
        }

        public static void Alert(string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger("alert");
            logger.Fatal(message, args);
        }

        public static void Alert(Exception ex, string message, params object[] args)
        {
            NLog.Logger logger = NLog.LogManager.GetLogger("alert");
            logger.Fatal(ex, message, args);
        }
    }
}

{% endhighlight %}

### 输出部分
输出部分主要体现在NLog.config配置如下：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">

  <!-- optional, add some variables
  https://github.com/nlog/NLog/wiki/Configuration-file#variables
  -->
  <variable name="myvar" value="myvalue"/>
  <variable name="ApplicationName" value="NLogUtility"/>
  <variable name="logDirectory" value="D:\itemlog\${ApplicationName}"/>
  <!--
  See https://github.com/nlog/nlog/wiki/Configuration-file
  for information on customizing logging rules and outputs.
   -->
  <targets>

    <!--
    add your targets here
    See https://github.com/nlog/NLog/wiki/Targets for possible targets.
    See https://github.com/nlog/NLog/wiki/Layout-Renderers for the possible layout renderers.
    -->
    <target xsi:type="ColoredConsole" name="console"
            layout="${time} ${message:exceptionSeparator=|:withException=true}${newline}*****************************" />
    <target xsi:type="File" name="error" encoding="utf-8" fileName="${logDirectory}\${shortdate}.log"
            layout="${time} ${message:exceptionSeparator=|:withException=true}${newline}*****************************" 
            archiveAboveSize="10485760" archiveNumbering="Sequence" />
    <target xsi:type="File" name="file" encoding="utf-8" fileName="${logDirectory}\${logger}_${shortdate}.log"
            layout="${time} ${uppercase:${level}} ${message}" archiveAboveSize="10485760" archiveNumbering="Sequence" />
    <target xsi:type="File" name="pure" encoding="utf-8" fileName="${logDirectory}\${logger}.log"
            layout="${message}" archiveAboveSize="10485760" archiveNumbering="Sequence" />
    <target xsi:type="Mail" name="alert" layout="${date}${newline}${message:exceptionSeparator=|:withException=true}${newline}"
            replaceNewlineWithBrTagInHtml="true" subject="Message from ${ApplicationName} on ${machinename}"
            smtpServer="smtp.exmail.qq.com" smtpAuthentication="Basic"
            to="receiver@xxx.com" from="sender@xxx.com" smtpUserName="sender@xxx.com" smtpPassword="senderPassword" />
    <!--
    Write events to a file with the date in the filename.
    <target xsi:type="File" name="f" fileName="${basedir}/logs/${shortdate}.log"
            layout="${longdate} ${uppercase:${level}} ${message}" />
    -->
  </targets>

  <rules>
    <!-- add your logging rules here -->
    <logger name="*" minlevel="Trace" writeTo="console" enabled="true" />
    <logger name="error" minlevel="Debug" writeTo="error" final="true" />
    <logger name="*.pure" minlevel="Trace" writeTo="pure" final="true" />
    <logger name="alert" minlevel="Debug" writeTo="alert" />
    <logger name="*" minlevel="Debug" writeTo="file" />
    <!--
    Write all events with minimal level of Debug (So Debug, Info, Warn, Error and Fatal, but not Trace)  to "f"
    <logger name="*" minlevel="Debug" writeTo="f" />
    -->
  </rules>
</nlog>
{% endhighlight %}

### 简单说明
1. 日志统一记录在D:\itemlog\NLogUtility下，建议将变量ApplicationName由NLogUtility改成你的应用程序的名字，D:\itemlog也可以更改成你想要的目录
2. 之所以将日志统一记录到D:\itemlog\目录下，主要是为了日后方便通过ftp工具查看线上服务器的日志，但有人习惯将日志记录到应用程序所在的目录下，那样可以将D:\itemlog\${ApplicationName}修为{basedir}
3. 默认不记录Trace级别的日志到文件（但会在Console输出），除非修改logger中的minlevel为Trace
4. 如果要实现将日志发送到邮箱的功能，需要配置有效的smtpServer，to， from，smtpUserName，smtpPassword


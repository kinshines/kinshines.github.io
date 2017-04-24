---
layout: post
title: 腾讯信鸽推送.NET SDK
author: kinshines
date:   2016-11-09
categories: c#
permalink: /archivers/xinge-push-sdk
---

<p class="lead"> <a href="http://xg.qq.com/xg">信鸽推送</a> 现改名为腾讯移动推送，是一款强大且免费的移动端推送工具，包括Android设备和iOS设备，官方提供的服务器端SDK有Java,php,python，本文将基于Rest API 实现.NET版SDK</p>

参考腾讯官方文档及资源：  
[信鸽基础介绍](http://developer.qq.com/wiki/xg/%E4%BF%A1%E9%B8%BD%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D/%E4%BF%A1%E9%B8%BD%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.html)  
[Rest API 使用指南](http://developer.qq.com/wiki/xg/%E6%9C%8D%E5%8A%A1%E7%AB%AFAPI%E6%8E%A5%E5%85%A5/Rest%20API%20%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/Rest%20API%20%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.html)  
[SDK 下载中心](http://xg.qq.com/xg/help/ctr_help/download)

完整代码参见：
[Github-XinGePush](https://github.com/kinshines/XinGePush)

快捷方式说明：
1. Android平台推送消息给单个设备
XingeApp.PushTokenAndroid(000, "myKey", "标题", "大家好!", "3dc4gcd98sdc");
2. Android平台推送消息给单个帐号
XingeApp.PushAccountAndroid(000, "myKey", "标题", "大家好!", "nickName");
3. Android平台推送消息给所有设备
XingeApp.PushAllAndroid(000, "myKey", "标题", "大家好!");
4. Android平台推送消息给标签选中设备
XingeApp.PushTagAndroid(000, "myKey", "标题", "大家好!", "beijing");
5. iOS平台推送消息给单个设备
XingeApp.PushTokenIos(000, "myKey", "你好!", "3dc4gcd98sdc", XingeApp.IOSENV_PROD);
6. iOS平台推送消息给单个帐号
XingeApp.PushAccountIos(000, "myKey", "你好", "nickName", XingeApp.IOSENV_PROD);
7. iOS平台推送消息给所有设备
XingeApp.PushAllIos(000, "myKey", "大家好!", XingeApp.IOSENV_PROD);
8. iOS平台推送消息给标签选中设备
XingeApp.PushTagIos(000, "myKey", "大家好!", "beijing", XingeApp.IOSENV_PROD);




---
layout: post
title: AndroidStudio 打包时 Signature Version 选择 V1 V2 说明
author: kinshines
date:   2017-08-31
categories: android
permalink: /archivers/android-studio-signature-version
---

<p class="lead">使用Android Studio ,打正式包的时候发现有两个签名版本选择：

<img src="https://kinshines.github.io/img/android-dev/signature_version.png"/>

因为刚开始默认勾选的v2(Full APK Signature)，结果在测试机上（4.4.2）一直都安装失败，想着和那个选择签名版本有关系，那就查查吧。</p>

### 问题描述(v1和v2)
Android 7.0中引入了APK Signature Scheme v2，v1是jar Signature来自JDK
V1：应该是通过ZIP条目进行验证，这样APK 签署后可进行许多修改 - 可以移动甚至重新压缩文件。
V2：验证压缩文件的所有字节，而不是单个 ZIP 条目，因此，在签名后无法再更改(包括 zipalign)。正因如此，现在在编译过程中，我们将压缩、调整和签署合并成一步完成。好处显而易见，更安全而且新的签名可缩短在设备上进行验证的时间（不需要费时地解压缩然后验证），从而加快应用安装速度。

### 解决方案一
v1和v2的签名使用
1. 只勾选v1签名并不会影响什么，但是在7.0上不会使用更安全的验证方式
2. 只勾选V2签名7.0以下会直接安装完显示未安装，7.0以上则使用了V2的方式验证
3. 同时勾选V1和V2则所有机型都没问题

### 解决方案二
在app的build.gradle的android标签下加入：
{% highlight gradle%}
signingConfigs {  
    debug {  
        v1SigningEnabled true  
        v2SigningEnabled true  
    }  
    release {  
        v1SigningEnabled true  
        v2SigningEnabled true  
    }  
}  
{% endhighlight %}
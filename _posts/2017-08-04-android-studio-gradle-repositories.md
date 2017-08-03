---
layout: post
title: AndroidStudio在build.gradle添加dependencies时提供repositories
author: kinshines
date:   2016-08-04
categories: android
permalink: /archivers/android-studio-gradle-dependencies-repositories
---

今天要需要在Android的项目里调用WebService，在app/build.gradle文件里添加[ksoap2-android](https://github.com/simpligility/ksoap2-android) dependencies

{% highlight js %}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:26+'
    compile 'com.google.code.ksoap2-android:ksoap2-android:3.6.2'
}

{% endhighlight %}

点击Sync，发现gradle无法找到该package，于是查询得知需要我告诉gradle下载地址，即添加repositories至app/build.gradle

{% highlight js %}

buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        repositories {
            maven { url 'https://oss.sonatype.org/content/repositories/ksoap2-android-releases' }
        }
    }

{% endhighlight %}

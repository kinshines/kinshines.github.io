---
layout: post
title: AndroidStudio 不能解析maven.google.com
author: kinshines
date:   2016-08-25
categories: android
permalink: /archivers/android-studio-resolve-maven-google-com
---

今天在用Android Studio gradle编译项目时，出现下面的错误

        compile "android.arch.lifecycle:runtime:1.0.0-alpha1"
        compile "android.arch.lifecycle:extensions:1.0.0-alpha1"
        annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha1"

因为上述jar包源自maven.google.com，这个站点在大陆被墙了，解决方案如下：将.gradle文件中的源url

        maven.google.com

替换为：

        https://dl.google.com/dl/android/maven2/


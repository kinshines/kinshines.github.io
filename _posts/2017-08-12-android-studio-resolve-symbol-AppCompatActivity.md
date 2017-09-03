---
layout: post
title: AndroidStudio在异常关机后报错：Cannot resolve symbol 'AppCompatActivity'
author: kinshines
date:   2016-08-12
categories: android
permalink: /archivers/android-studio-resolve-symbol-AppCompatActivity
---

今天在用Android Studio 调试项目时，因为电脑CPU过热而死机，强制重启后，重新打开项目后Android Studio 显示下面的错误

        Cannot resolve symbol 'AppCompatActivity'

刚接触Android Studio，无奈借助google搜索，找了一大堆解决方案后，最简洁有效的方法如下：

        File->Invalidate Caches/Restart

![Invalidate Caches/Restart](https://kinshines.github.io/img/android-dev/Invalidate_Caches.png)

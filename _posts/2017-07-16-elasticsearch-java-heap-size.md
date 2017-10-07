---
layout: post
title: Elasticsearch修改最大Java堆大小
author: kinshines
date:   2017-07-16
categories: db
permalink: /archivers/elasticsearch-java-heap-size
---

<p class="lead">今天ES搜索在服务器上莫名奇妙地自动停止，检查ES主目录下生成了一个hs_err_pid.log文件，记录了详细的错误信息：</p>

{% highlight js %}
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (malloc) failed to allocate 32744 bytes for ChunkPool::allocate
# Possible reasons:
#   The system is out of physical RAM or swap space
#   In 32 bit mode, the process size limit was hit
# Possible solutions:
#   Reduce memory load on the system
#   Increase physical memory or swap space
#   Check if swap backing store is full
#   Use 64 bit Java on a 64 bit OS
#   Decrease Java heap size (-Xmx/-Xms)
#   Decrease number of Java threads
#   Decrease Java thread stack sizes (-Xss)
#   Set larger code cache with -XX:ReservedCodeCacheSize=
# This output file may be truncated or incomplete.
#
#  Out of Memory Error (allocation.cpp:273), pid=14604, tid=0x0000000000002f3c
#
# JRE version: Java(TM) SE Runtime Environment (8.0_112-b15) (build 1.8.0_112-b15)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.112-b15 mixed mode windows-amd64 compressed oops)
# Failed to write core dump. 
#

---------------  T H R E A D  ---------------

Current thread (0x0000000011fb5800):  JavaThread "C2 CompilerThread0" daemon [_thread_in_native, id=12092, stack(0x0000000012150000,0x0000000012250000)]

Stack: [0x0000000012150000,0x0000000012250000]
[error occurred during error reporting (printing stack bounds), id 0xc0000005]

Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)


Current CompileTask:
C2:  45134 6823 %     4       org.apache.lucene.codecs.lucene54.Lucene54DocValuesConsumer::addNumericField @ 237 (1438 bytes)


---------------  P R O C E S S  ---------------

Java Threads: ( => current thread )
.
.
.
VM Arguments:
jvm_args: -XX:+UseParNewGC -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Delasticsearch -Des.path.home=D:\ES\elasticsearch -Des.default.path.home=D:\ES\elasticsearch -Des.default.path.logs=D:\ES\elasticsearch\logs -Des.default.path.data=D:\ES\elasticsearch\data -Des.default.path.conf=D:\ES\elasticsearch\config exit -Xms256m -Xmx1024m -Xss256k 
java_command: <unknown>
.
.
.
{% endhighlight %}

当前服务器内存为8G，使用率为50%，所以内存应该是足够用的，从log日志中得到当前Java max heap size 为1024m，根据提示中的Possible Solution，尝试降低Java heap size，使用cmd窗口输入命令

        java -Xmx512m

重新启动ES，问题解决



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

今天升级ElasticSearch至5.5版本，发现config目录下相比于2.2版本多了一个名为jvm.options的配置文件，里边记录了关于jvm的配置信息，因为以上解决方案针对的是服务器层面的配置修改，难免会影响到其他的程序，因此，我们可以在jvm.options里修改以上配置，达到同样的效果。

先看一下默认的jvm.options文件内容：

{% highlight js %}
## JVM configuration

################################################################
## IMPORTANT: JVM heap size
################################################################
##
## You should always set the min and max JVM heap
## size to the same value. For example, to set
## the heap to 4 GB, set:
##
## -Xms4g
## -Xmx4g
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html
## for more information
##
################################################################

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms2g
-Xmx2g

################################################################
## Expert settings
################################################################
##
## All settings below this section are considered
## expert settings. Don't tamper with them unless
## you understand what you are doing
##
################################################################

## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly

## optimizations

# disable calls to System#gc
-XX:+DisableExplicitGC

# pre-touch memory pages used by the JVM during initialization
-XX:+AlwaysPreTouch

## basic

# force the server VM (remove on 32-bit client JVMs)
-server

# explicitly set the stack size (reduce to 320k on 32-bit client JVMs)
-Xss1m

# set to headless, just in case
-Djava.awt.headless=true

# ensure UTF-8 encoding by default (e.g. filenames)
-Dfile.encoding=UTF-8

# use our provided JNA always versus the system one
-Djna.nosys=true

# use old-style file permissions on JDK9
-Djdk.io.permissionsUseCanonicalPath=true

# flags to configure Netty
-Dio.netty.noUnsafe=true
-Dio.netty.noKeySetOptimization=true
-Dio.netty.recycler.maxCapacityPerThread=0

# log4j 2
-Dlog4j.shutdownHookEnabled=false
-Dlog4j2.disable.jmx=true
-Dlog4j.skipJansi=true

## heap dumps

# generate a heap dump when an allocation from the Java heap fails
# heap dumps are created in the working directory of the JVM
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps
# ensure the directory exists and has sufficient space
#-XX:HeapDumpPath=${heap.dump.path}

## GC logging

#-XX:+PrintGCDetails
#-XX:+PrintGCTimeStamps
#-XX:+PrintGCDateStamps
#-XX:+PrintClassHistogram
#-XX:+PrintTenuringDistribution
#-XX:+PrintGCApplicationStoppedTime

# log GC status to a file with time stamps
# ensure the directory exists
#-Xloggc:${loggc}

# By default, the GC log file will not rotate.
# By uncommenting the lines below, the GC log file
# will be rotated every 128MB at most 32 times.
#-XX:+UseGCLogFileRotation
#-XX:NumberOfGCLogFiles=32
#-XX:GCLogFileSize=128M

# Elasticsearch 5.0.0 will throw an exception on unquoted field names in JSON.
# If documents were already indexed with unquoted fields in a previous version
# of Elasticsearch, some operations may throw errors.
#
# WARNING: This option will be removed in Elasticsearch 6.0.0 and is provided
# only for migration purposes.
#-Delasticsearch.json.allow_unquoted_field_names=true

{% endhighlight %}

将第22，23行修改为：

        -Xms512m
        -Xmx512m

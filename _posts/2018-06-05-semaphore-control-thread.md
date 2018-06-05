---
layout: post
title: 使用Semaphore（信号量）控制多线程
author: kinshines
date:   2018-06-05
categories: c#
permalink: /archivers/semaphore-control-thread
---

<p class="lead">在.NET中，当需要动态控制线程的数量时，我们可以使用Semaphore来控制最大线程数。
</p>

### 关于Semaphore代码示例

{% highlight java %}

 class Program
 {
         //Semaphore（初始授予0个请求数，设置最大可授予5个请求数）
         static Semaphore semaphore = new Semaphore(0, 5);

         static void Main(string[] args)
         {
             for (int i = 1; i <= 5; i++)
             {
                 Thread thread = new Thread(work);
                 thread.Start(i);
             } 

             Thread.Sleep(1000);
             Console.WriteLine("Main方法结束");

             //授予5个请求
             semaphore.Release(5);
             Console.ReadLine();
         }

         static void work(object obj)
         {
             semaphore.WaitOne();
             Console.WriteLine("print: {0}", obj);
             semaphore.Release();
         }
 }

{% endhighlight %}

### 代码说明
 new Semaphore(0, 5); 构造函数第一个参数，表示我们还可使用的授权数。 第二个参数表示我们最大可申请的授权数。


当授权数用完时，则会造成线程阻塞直到可申请到Semaphore的授权。所以如上代码我一开始初始化了0个授权数，所以没有授权则会被阻塞。


在main方法快运行完时，我使用代码semaphore.Release(5);授权了5个请求。 这时还阻塞在semaphore.WaitOne();的代码得到授权则开始继续往下运行，打印出print:{0} 。
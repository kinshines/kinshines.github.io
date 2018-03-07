---
layout: post
title: 使用Wait()和GetAwaiter().GetResult()方法实现异步方法同步执行
author: kinshines
date:   2018-03-07
categories: c#
permalink: /archivers/sync-invoke-async-method
---

<p class="lead">使用 await 和 async 关键字能够非常方便的解决异步调用的问题，但在某些业务场景中，我们需要保证方法执行的顺序，所以需要同步执行异步方法，下面提供解决方案</p>

对于无需返回值的异步方法，只需在异步方法后面加Wait()方法即可

对于需要返回值的异步方法，需要在异步方法后面追加GetAwaiter().GetResult()

{% highlight java %}

using System;
using System.Threading.Tasks;

namespace AsyncTest
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Async Test job:");

            Console.WriteLine("main start..");

            Console.WriteLine("MyMethod()异步方法同步执行：");

            MyMethod().Wait();//在main中等待async方法执行完成

            int i = MyMethod().GetAwaiter().GetResult();//用于在main中同步获取async方法的返回结果

            Console.WriteLine("i:" + i);

            Console.WriteLine("main end..");

            Console.ReadKey(true);
        }

        static async Task<int> MyMethod()
        {
            for (int i = 0; i < 5; i++)
            {
                Console.WriteLine("Async start:" + i.ToString() + "..");
                await Task.Delay(1000); //模拟耗时操作
            }
            return 0;
        }
    }
}

{% endhighlight %}
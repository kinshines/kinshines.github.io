---
layout: post
title: 单例模式
author: kinshines
date:   2018-05-23
categories: design-pattern
permalink: /archivers/design-pattern-singleton
---

<p class="lead">
单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
<br/>
这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
</p>

### 意图
保证一个类仅有一个实例，并提供一个访问它的全局访问点。
### 主要解决
一个全局使用的类频繁地创建与销毁。
### 何时使用
当您想控制实例数目，节省系统资源的时候。
### 如何解决
判断系统是否已经有这个单例，如果有则返回，如果没有则创建。
### 关键代码
构造函数是私有的。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Singleton s1 = Singleton.GetInstance();
            Singleton s2 = Singleton.GetInstance();

            if (s1 == s2)
            {
                Console.WriteLine("Objects are the same instance");
            }

            Console.Read();
        }
    }


    class Singleton
    {
        private static Singleton instance;
        private static readonly object syncRoot = new object();
        private Singleton()
        {
        }

        // 首次访问时加载，懒汉式
        public static Singleton GetInstance()
        {
            if (instance == null)
            {
                lock (syncRoot)
                {
                    if (instance == null)
                    {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }

    // 程序启动时初始化，饿汉式
    public sealed class Singleton
    {
        private static readonly Singleton instance = new Singleton();
        private Singleton() { }
        public static Singleton GetInstance()
        {
            return instance;
        }
    }

{% endhighlight %}
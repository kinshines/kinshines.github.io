---
layout: post
title: 代理模式
author: kinshines
date:   2018-05-16
categories: design-pattern
permalink: /archivers/design-pattern-proxy
---

<p class="lead">在代理模式（Proxy Pattern）中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式。
<br/>
在代理模式中，我们创建具有现有对象的对象，以便向外界提供功能接口。
</p>

### 意图
为其他对象提供一种代理以控制对这个对象的访问。
### 主要解决
在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。
### 何时使用
想在访问一个类时做一些控制。
### 如何解决
增加中间层。
### 关键代码
实现与被代理类组合。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Proxy proxy = new Proxy();
            proxy.Request();
            Console.Read();
        }
    }

    abstract class Subject
    {
        public abstract void Request();
    }

    class RealSubject : Subject
    {
        public override void Request()
        {
            Console.WriteLine("真实的请求");
        }
    }

    class Proxy : Subject
    {
        RealSubject realSubject;
        public override void Request()
        {
            if (realSubject == null)
            {
                realSubject = new RealSubject();
            }

            realSubject.Request();
        }
    }

{% endhighlight %}

### 示例
代理给女生送礼物

#### 步骤1 创建一个接口

{% highlight java %}

    //送礼物
    interface GiveGift
    {
        void GiveDolls();
        void GiveFlowers();
        void GiveChocolate();
    }

{% endhighlight %}

#### 步骤2 实现接口的实体类和代理类

{% highlight java %}

    class Proxy : GiveGift
    {
        Pursuit gg;
        public Proxy(SchoolGirl mm)
        {
            gg = new Pursuit(mm);
        }


        public void GiveDolls()
        {
            gg.GiveDolls();
        }

        public void GiveFlowers()
        {
            gg.GiveFlowers();
        }

        public void GiveChocolate()
        {
            gg.GiveChocolate();
        }
    }

    class Pursuit : GiveGift
    {
        SchoolGirl mm;
        public Pursuit(SchoolGirl mm)
        {
            this.mm = mm;
        }
        public void GiveDolls()
        {
            Console.WriteLine(mm.Name + " 送你洋娃娃");
        }

        public void GiveFlowers()
        {
            Console.WriteLine(mm.Name + " 送你鲜花");
        }

        public void GiveChocolate()
        {
            Console.WriteLine(mm.Name + " 送你巧克力");
        }
    }

    class SchoolGirl
    {
        private string name;
        public string Name
        {
            get { return name; }
            set { name = value; }
        }
    }

{% endhighlight %}

#### 步骤3 使用代理来访问对象。

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            SchoolGirl jiaojiao = new SchoolGirl();
            jiaojiao.Name = "李娇娇";

            Proxy daili = new Proxy(jiaojiao);

            daili.GiveDolls();
            daili.GiveFlowers();
            daili.GiveChocolate();

            Console.Read();
        }
    }

{% endhighlight %}
---
layout: post
title: 桥接模式
author: kinshines
date:   2018-05-23
categories: design-pattern
permalink: /archivers/design-pattern-bridge
---

<p class="lead">
桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。
<br/>
这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。
</p>

### 意图
将抽象部分与实现部分分离，使它们都可以独立的变化。
### 主要解决
在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活。
### 何时使用
实现系统可能有多个角度分类，每一种角度都可能变化。
### 如何解决
把这种多角度分类分离出来，让它们独立变化，减少它们之间耦合。
### 关键代码
抽象类依赖实现类。

### 基本代码

{% highlight java %}

       class Program
    {
        static void Main(string[] args)
        {
            Abstraction ab = new RefinedAbstraction();

            ab.SetImplementor(new ConcreteImplementorA());
            ab.Operation();

            ab.SetImplementor(new ConcreteImplementorB());
            ab.Operation();

            Console.Read();
        }
    }

    class Abstraction
    {
        protected Implementor implementor;

        public void SetImplementor(Implementor implementor)
        {
            this.implementor = implementor;
        }

        public virtual void Operation()
        {
            implementor.Operation();
        }

    }

    class RefinedAbstraction : Abstraction
    {
        public override void Operation()
        {
            implementor.Operation();
        }
    }

    abstract class Implementor
    {
        public abstract void Operation();
    }

    class ConcreteImplementorA : Implementor
    {
        public override void Operation()
        {
            Console.WriteLine("具体实现A的方法执行");
        }
    }

    class ConcreteImplementorB : Implementor
    {
        public override void Operation()
        {
            Console.WriteLine("具体实现B的方法执行");
        }
    }

{% endhighlight %}

### 示例
不同的手机品牌运行软件

#### 步骤1 创建抽象类

{% highlight java %}

     //手机品牌
    abstract class HandsetBrand
    {
        protected HandsetSoft soft;

        //设置手机软件
        public void SetHandsetSoft(HandsetSoft soft)
        {
            this.soft = soft;
        }
        //运行
        public abstract void Run();
    }

{% endhighlight %}

#### 步骤2 实现抽象类的具体类

{% highlight java %}

     //手机品牌N
    class HandsetBrandN : HandsetBrand
    {
        public override void Run()
        {
            soft.Run();
        }
    }

    //手机品牌M
    class HandsetBrandM : HandsetBrand
    {
        public override void Run()
        {
            soft.Run();
        }
    }

    //手机品牌S
    class HandsetBrandS : HandsetBrand
    {
        public override void Run()
        {
            soft.Run();
        }
    }

{% endhighlight %}


#### 步骤3 创建抽象功能

{% highlight java %}

     //手机软件
    abstract class HandsetSoft
    {

        public abstract void Run();
    }

{% endhighlight %}

#### 步骤4 实现具体功能

{% highlight java %}

     //手机游戏
    class HandsetGame : HandsetSoft
    {
        public override void Run()
        {
            Console.WriteLine("运行手机游戏");
        }
    }

    //手机通讯录
    class HandsetAddressList : HandsetSoft
    {
        public override void Run()
        {
            Console.WriteLine("运行手机通讯录");
        }
    }

    //手机MP3播放
    class HandsetMP3 : HandsetSoft
    {
        public override void Run()
        {
            Console.WriteLine("运行手机MP3播放");
        }
    }

{% endhighlight %}

#### 步骤5 使用桥接实现不同类运行功能

{% highlight java %}

    static void Main(string[] args)
        {
            HandsetBrand ab;
            ab = new HandsetBrandN();

            ab.SetHandsetSoft(new HandsetGame());
            ab.Run();

            ab.SetHandsetSoft(new HandsetAddressList());
            ab.Run();

            ab = new HandsetBrandM();

            ab.SetHandsetSoft(new HandsetGame());
            ab.Run();

            ab.SetHandsetSoft(new HandsetAddressList());
            ab.Run();

            Console.Read();
        }

{% endhighlight %}
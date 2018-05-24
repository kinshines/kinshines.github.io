---
layout: post
title: 享元模式
author: kinshines
date:   2018-05-24
categories: design-pattern
permalink: /archivers/design-pattern-flyweight
---

<p class="lead">
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。
</p>

### 意图
运用共享技术有效地支持大量细粒度的对象。
### 主要解决
在有大量对象时，有可能会造成内存溢出，我们把其中共同的部分抽象出来，如果有相同的业务请求，直接返回在内存中已有的对象，避免重新创建。
### 何时使用
1. 系统中有大量对象。 
2. 这些对象消耗大量内存。 
3. 这些对象的状态大部分可以外部化。 
4. 这些对象可以按照内蕴状态分为很多组，当把外蕴对象从对象中剔除出来时，每一组对象都可以用一个对象来代替。 
5. 系统不依赖于这些对象身份，这些对象是不可分辨的。
### 如何解决
用唯一标识码判断，如果在内存中有，则返回这个唯一标识码所标识的对象。
### 关键代码
用 HashMap 存储这些对象。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            int extrinsicstate = 22;

            FlyweightFactory f = new FlyweightFactory();

            Flyweight fx = f.GetFlyweight("X");
            fx.Operation(--extrinsicstate);

            Flyweight fy = f.GetFlyweight("Y");
            fy.Operation(--extrinsicstate);

            Flyweight fz = f.GetFlyweight("Z");
            fz.Operation(--extrinsicstate);

            UnsharedConcreteFlyweight uf = new UnsharedConcreteFlyweight();

            uf.Operation(--extrinsicstate);

            Console.Read();
        }
    }

    class FlyweightFactory
    {
        private Hashtable flyweights = new Hashtable();

        public FlyweightFactory()
        {
            flyweights.Add("X", new ConcreteFlyweight());
            flyweights.Add("Y", new ConcreteFlyweight());
            flyweights.Add("Z", new ConcreteFlyweight());

        }

        public Flyweight GetFlyweight(string key)
        {
            return ((Flyweight)flyweights[key]);
        }
    }

    abstract class Flyweight
    {
        public abstract void Operation(int extrinsicstate);
    }

    class ConcreteFlyweight : Flyweight
    {
        public override void Operation(int extrinsicstate)
        {
            Console.WriteLine("具体Flyweight:" + extrinsicstate);
        }
    }

    class UnsharedConcreteFlyweight : Flyweight
    {
        public override void Operation(int extrinsicstate)
        {
            Console.WriteLine("不共享的具体Flyweight:" + extrinsicstate);
        }
    }

{% endhighlight %}

### 示例
用户使用网站空间

#### 步骤1 创建抽象类

{% highlight java %}

    //网站
    abstract class WebSite
    {
        public abstract void Use(User user);
    }

        //用户
    public class User
    {
        private string name;

        public User(string name)
        {
            this.name = name;
        }

        public string Name
        {
            get { return name; }
        }
    }

{% endhighlight %}

#### 步骤2 创建具体类

{% highlight java %}

    //具体的网站
    class ConcreteWebSite : WebSite
    {
        private string name = "";
        public ConcreteWebSite(string name)
        {
            this.name = name;
        }

        public override void Use(User user)
        {
            Console.WriteLine("网站分类：" + name + " 用户：" + user.Name);
        }
    }

{% endhighlight %}

#### 步骤3 创建具体类的管理工厂

{% highlight java %}

        //网站工厂
    class WebSiteFactory
    {
        private Hashtable flyweights = new Hashtable();

        //获得网站分类
        public WebSite GetWebSiteCategory(string key)
        {
            if (!flyweights.ContainsKey(key))
                flyweights.Add(key, new ConcreteWebSite(key));
            return ((WebSite)flyweights[key]);
        }

        //获得网站分类总数
        public int GetWebSiteCount()
        {
            return flyweights.Count;
        }
    }

{% endhighlight %}

#### 步骤2 实现抽象类的具体类

{% highlight java %}

     //联合国安全理事会
    class UnitedNationsSecurityCouncil : UnitedNations
    {
        private USA colleague1;
        private Iraq colleague2;

        public USA Colleague1
        {
            set { colleague1 = value; }
        }

        public Iraq Colleague2
        {
            set { colleague2 = value; }
        }

        public override void Declare(string message, Country colleague)
        {
            if (colleague == colleague1)
            {
                colleague2.GetMessage(message);
            }
            else
            {
                colleague1.GetMessage(message);
            }
        }
    }

{% endhighlight %}


#### 步骤3 创建抽象通信类

{% highlight java %}

      //国家
    abstract class Country
    {
        protected UnitedNations mediator;

        public Country(UnitedNations mediator)
        {
            this.mediator = mediator;
        }
    }

{% endhighlight %}

#### 步骤4 实现享元

{% highlight java %}

    static void Main(string[] args)
        {

            WebSiteFactory f = new WebSiteFactory();

            WebSite fx = f.GetWebSiteCategory("产品展示");
            fx.Use(new User("小菜"));

            WebSite fy = f.GetWebSiteCategory("产品展示");
            fy.Use(new User("大鸟"));

            WebSite fz = f.GetWebSiteCategory("产品展示");
            fz.Use(new User("娇娇"));

            WebSite fl = f.GetWebSiteCategory("博客");
            fl.Use(new User("老顽童"));

            WebSite fm = f.GetWebSiteCategory("博客");
            fm.Use(new User("桃谷六仙"));

            WebSite fn = f.GetWebSiteCategory("博客");
            fn.Use(new User("南海鳄神"));

            Console.WriteLine("得到网站分类总数为 {0}", f.GetWebSiteCount());

            //string titleA = "大话设计模式";
            //string titleB = "大话设计模式";

            //Console.WriteLine(Object.ReferenceEquals(titleA, titleB));


            Console.Read();
        }

{% endhighlight %}

### 后记
如果一个应用程序使用了大量的对象，而大量的这些对象造成了很大的存储开销时应该考虑使用享元模式，还有就是对象的大多数状态可以是外部状态，如果删除对象的外部状态，那么可以用相对较少的共享对象取代很多组对象，此时可以考虑使用享元模式。
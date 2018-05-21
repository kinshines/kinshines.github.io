---
layout: post
title: 适配器模式
author: kinshines
date:   2018-05-21
categories: design-pattern
permalink: /archivers/design-pattern-adapter
---

<p class="lead">
适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。
<br/>
这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能
</p>

### 意图
将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
### 主要解决
主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。
### 何时使用
 1. 系统需要使用现有的类，而此类的接口不符合系统的需要。 
 2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。 
 3. 通过接口转换，将一个类插入另一个类系中。

### 如何解决
继承或依赖（推荐）。
### 关键代码
适配器继承或依赖已有的对象，实现想要的目标接口。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Target target = new Adapter();
            target.Request();

            Console.Read();

        }
    }

    class Target
    {
        public virtual void Request()
        {
            Console.WriteLine("普通请求");
        }
    }

    class Adaptee
    {
        public void SpecificRequest()
        {
            Console.WriteLine("特殊请求");
        }
    }

    class Adapter : Target
    {
        private Adaptee adaptee = new Adaptee();

        public override void Request()
        {
            adaptee.SpecificRequest();
        }
    }

{% endhighlight %}

### 示例
姚明去NBA打篮球，需要翻译

#### 步骤1 创建抽象类

{% highlight java %}

     //篮球运动员
    abstract class Player
    {
        protected string name;
        public Player(string name)
        {
            this.name = name;
        }

        public abstract void Attack();
        public abstract void Defense();
    }

{% endhighlight %}

#### 步骤2 具体类。

{% highlight java %}

    //前锋
    class Forwards : Player
    {
        public Forwards(string name)
            : base(name)
        {
        }

        public override void Attack()
        {
            Console.WriteLine("前锋 {0} 进攻", name);
        }

        public override void Defense()
        {
            Console.WriteLine("前锋 {0} 防守", name);
        }
    }

    //中锋
    class Center : Player
    {
        public Center(string name)
            : base(name)
        {
        }

        public override void Attack()
        {
            Console.WriteLine("中锋 {0} 进攻", name);
        }

        public override void Defense()
        {
            Console.WriteLine("中锋 {0} 防守", name);
        }
    }

    //后卫
    class Guards : Player
    {
        public Guards(string name)
            : base(name)
        {
        }

        public override void Attack()
        {
            Console.WriteLine("后卫 {0} 进攻", name);
        }

        public override void Defense()
        {
            Console.WriteLine("后卫 {0} 防守", name);
        }
    }

    //外籍中锋
    class ForeignCenter
    {
        private string name;
        public string Name
        {
            get { return name; }
            set { name = value; }
        }

        public void 进攻()
        {
            Console.WriteLine("外籍中锋 {0} 进攻", name);
        }

        public void 防守()
        {
            Console.WriteLine("外籍中锋 {0} 防守", name);
        }
    }

{% endhighlight %}

#### 步骤3 为需要兼容的类创建适配器。

{% highlight java %}

    //翻译者
    class Translator : Player
    {
        private ForeignCenter wjzf = new ForeignCenter();

        public Translator(string name)
            : base(name)
        {
            wjzf.Name = name;
        }

        public override void Attack()
        {
            wjzf.进攻();
        }

        public override void Defense()
        {
            wjzf.防守();
        }
    }

{% endhighlight %}

#### 步骤4 对外使用同一接口调用

{% highlight java %}

    static void Main(string[] args)
        {
            Player b = new Forwards("巴蒂尔");
            b.Attack();

            Player m = new Guards("麦克格雷迪");
            m.Attack();

            //Player ym = new Center("姚明");
            Player ym = new Translator("姚明");
            ym.Attack();
            ym.Defense();

            Console.Read();
        }

{% endhighlight %}
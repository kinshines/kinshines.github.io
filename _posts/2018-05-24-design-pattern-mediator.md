---
layout: post
title: 中介者模式
author: kinshines
date:   2018-05-24
categories: design-pattern
permalink: /archivers/design-pattern-mediator
---

<p class="lead">
中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。
<br/>
中介者模式属于行为型模式。
</p>

### 意图
用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。
### 主要解决
对象与对象之间存在大量的关联关系，这样势必会导致系统的结构变得很复杂，同时若一个对象发生改变，我们也需要跟踪与之相关联的对象，同时做出相应的处理。
### 何时使用
多个类相互耦合，形成了网状结构。
### 如何解决
将上述网状结构分离为星型结构。
### 关键代码
对象 Colleague 之间的通信封装到一个类中单独处理。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            ConcreteMediator m = new ConcreteMediator();

            ConcreteColleague1 c1 = new ConcreteColleague1(m);
            ConcreteColleague2 c2 = new ConcreteColleague2(m);

            m.Colleague1 = c1;
            m.Colleague2 = c2;

            c1.Send("吃过饭了吗?");
            c2.Send("没有呢，你打算请客？");

            Console.Read();
        }
    }

    abstract class Mediator
    {
        public abstract void Send(string message, Colleague colleague);
    }

    class ConcreteMediator : Mediator
    {
        private ConcreteColleague1 colleague1;
        private ConcreteColleague2 colleague2;

        public ConcreteColleague1 Colleague1
        {
            set { colleague1 = value; }
        }

        public ConcreteColleague2 Colleague2
        {
            set { colleague2 = value; }
        }

        public override void Send(string message, Colleague colleague)
        {
            if (colleague == colleague1)
            {
                colleague2.Notify(message);
            }
            else
            {
                colleague1.Notify(message);
            }
        }
    }

    abstract class Colleague
    {
        protected Mediator mediator;

        public Colleague(Mediator mediator)
        {
            this.mediator = mediator;
        }
    }

    class ConcreteColleague1 : Colleague
    {
        public ConcreteColleague1(Mediator mediator)
            : base(mediator)
        {

        }

        public void Send(string message)
        {
            mediator.Send(message, this);
        }

        public void Notify(string message)
        {
            Console.WriteLine("同事1得到信息:" + message);
        }
    }

    class ConcreteColleague2 : Colleague
    {
        public ConcreteColleague2(Mediator mediator)
            : base(mediator)
        {
        }

        public void Send(string message)
        {
            mediator.Send(message, this);
        }

        public void Notify(string message)
        {
            Console.WriteLine("同事2得到信息:" + message);
        }
    }

{% endhighlight %}

### 示例
国家之间通过联合国对话

#### 步骤1 创建抽象的中介者

{% highlight java %}

     //联合国机构
    abstract class UnitedNations
    {
        /// <summary>
        /// 声明
        /// </summary>
        /// <param name="message">声明信息</param>
        /// <param name="colleague">声明国家</param>
        public abstract void Declare(string message, Country colleague);
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

#### 步骤4 实现具体通信类

{% highlight java %}

      //美国
    class USA : Country
    {
        public USA(UnitedNations mediator)
            : base(mediator)
        {

        }
        //声明
        public void Declare(string message)
        {
            mediator.Declare(message, this);
        }
        //获得消息
        public void GetMessage(string message)
        {
            Console.WriteLine("美国获得对方信息：" + message);
        }
    }

    //伊拉克
    class Iraq : Country
    {
        public Iraq(UnitedNations mediator)
            : base(mediator)
        {
        }

        //声明
        public void Declare(string message)
        {
            mediator.Declare(message, this);
        }
        //获得消息
        public void GetMessage(string message)
        {
            Console.WriteLine("伊拉克获得对方信息：" + message);
        }

    }

{% endhighlight %}

#### 步骤5 使用中介者实现通信

{% highlight java %}

    static void Main(string[] args)
        {
            UnitedNationsSecurityCouncil UNSC = new UnitedNationsSecurityCouncil();

            USA c1 = new USA(UNSC);
            Iraq c2 = new Iraq(UNSC);

            UNSC.Colleague1 = c1;
            UNSC.Colleague2 = c2;

            c1.Declare("不准研制核武器，否则要发动战争！");
            c2.Declare("我们没有核武器，也不怕侵略。");

            Console.Read();
        }

{% endhighlight %}

### 后记
中介者模式很容易在系统中应用，也很容易在系统中误用。当系统出现了“多对多”交互复杂的对象群时，不要急于使用中介者模式，而是先反思你的系统在设计上是不是合理。

Mediator的出现减少了各个Colleague的耦合，使得可以独立地改变和复用各个Colleague类和Mediator，由于把对象如何协作进行了抽象，将中介作为一个独立的概念并将其封装在一个对象中，这样关注的对象就从对象各自本身的行为转移到它们之间的交互上来，也就是站在一个更宏观的角度去看待系统。

由于ConcreteMediator 控制了集中化，于是就把交互复杂性变为了中介者的复杂性，这就使得中介者会变得比任何一个ConcreteColleague 都复杂

WinForm中的Form和WebForm中的aspx都是典型的中介者

中介者模式一般应用于一组对象以定义良好但是复杂的方式进行通信的场合，比如WinForm中的Form和WebForm中的aspx，以及想定制一个分布在多个类中的行为，而又不想生成太多子类的场合。
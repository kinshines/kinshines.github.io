---
layout: post
title: 观察者模式
author: kinshines
date:   2018-05-19
categories: design-pattern
permalink: /archivers/design-pattern-observer
---

<p class="lead">
当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知它的依赖对象。
<br/>
观察者模式属于行为型模式。
</p>

### 意图
定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
### 主要解决
一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。
### 何时使用
一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。
### 如何解决
使用面向对象技术，可以将这种依赖关系弱化。
### 关键代码
在抽象类里有一个 ArrayList 存放观察者们。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            ConcreteSubject s = new ConcreteSubject();

            s.Attach(new ConcreteObserver(s, "X"));
            s.Attach(new ConcreteObserver(s, "Y"));
            s.Attach(new ConcreteObserver(s, "Z"));

            s.SubjectState = "ABC";
            s.Notify();

            Console.Read();
        }
    }


    abstract class Subject
    {
        private IList<Observer> observers = new List<Observer>();

        //增加观察者
        public void Attach(Observer observer)
        {
            observers.Add(observer);
        }
        //移除观察者
        public void Detach(Observer observer)
        {
            observers.Remove(observer);
        }
        //通知
        public void Notify()
        {
            foreach (Observer o in observers)
            {
                o.Update();
            }
        }
    }

    //具体通知者
    class ConcreteSubject : Subject
    {
        private string subjectState;

        //具体通知者状态
        public string SubjectState
        {
            get { return subjectState; }
            set { subjectState = value; }
        }
    }


    abstract class Observer
    {
        public abstract void Update();
    }

    class ConcreteObserver : Observer
    {
        private string name;
        private string observerState;
        private ConcreteSubject subject;

        public ConcreteObserver(
          ConcreteSubject subject, string name)
        {
            this.subject = subject;
            this.name = name;
        }
        //更新
        public override void Update()
        {
            observerState = subject.SubjectState;
            Console.WriteLine("观察者{0}的新状态是{1}",
              name, observerState);
        }

        public ConcreteSubject Subject
        {
            get { return subject; }
            set { subject = value; }
        }
    }

{% endhighlight %}

### 示例
老板外出回来了，通知各职员回到工作状态

#### 步骤1 创建通知者接口

{% highlight java %}

     //通知者接口
    interface Subject
    {
        void Notify();
        string SubjectState
        {
            get;
            set;
        }
    }

{% endhighlight %}

#### 步骤2 实现通知者类。

{% highlight java %}

    //事件处理程序的委托
    delegate void EventHandler();

    class Secretary : Subject
    {
        //声明一事件Update，类型为委托EventHandler
        public event EventHandler Update;

        private string action;

        public void Notify()
        {
            Update();
        }
        public string SubjectState
        {
            get { return action; }
            set { action = value; }
        }
    }

    class Boss : Subject
    {
        //声明一事件Update，类型为委托EventHandler
        public event EventHandler Update;

        private string action;

        public void Notify()
        {
            Update();
        }
        public string SubjectState
        {
            get { return action; }
            set { action = value; }
        }
    }

{% endhighlight %}

#### 步骤3 创建观察者类。

{% highlight java %}

    //看股票的同事
    class StockObserver
    {
        private string name;
        private Subject sub;
        public StockObserver(string name, Subject sub)
        {
            this.name = name;
            this.sub = sub;
        }

        //关闭股票行情
        public void CloseStockMarket()
        {
            Console.WriteLine("{0} {1} 关闭股票行情，继续工作！", sub.SubjectState, name);
        }
    }

    //看NBA的同事
    class NBAObserver
    {
        private string name;
        private Subject sub;
        public NBAObserver(string name, Subject sub)
        {
            this.name = name;
            this.sub = sub;
        }

        //关闭NBA直播
        public void CloseNBADirectSeeding()
        {
            Console.WriteLine("{0} {1} 关闭NBA直播，继续工作！", sub.SubjectState, name);
        }
    }

{% endhighlight %}

#### 步骤4 使用委托通知各观察者

{% highlight java %}

        static void Main(string[] args)
        {
            //老板胡汉三
            Boss huhansan = new Boss();

            //看股票的同事
            StockObserver tongshi1 = new StockObserver("魏关姹", huhansan);
            //看NBA的同事
            NBAObserver tongshi2 = new NBAObserver("易管查", huhansan);

            huhansan.Update += new EventHandler(tongshi1.CloseStockMarket);
            huhansan.Update += new EventHandler(tongshi2.CloseNBADirectSeeding);

            //老板回来
            huhansan.SubjectState = "我胡汉三回来了！";
            //发出通知
            huhansan.Notify();

            Console.Read();
        }

{% endhighlight %}
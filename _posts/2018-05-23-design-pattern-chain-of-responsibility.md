---
layout: post
title: 职责链模式
author: kinshines
date:   2018-05-23
categories: design-pattern
permalink: /archivers/design-pattern-chain-of-responsibility
---

<p class="lead">
顾名思义，责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。

<br />
在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。
</p>

### 意图
避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。
### 主要解决
职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。
### 何时使用
在处理消息的时候以过滤很多道。
### 如何解决
拦截的类都实现统一接口。
### 关键代码
Handler 里面聚合它自己，在 HanleRequest 里判断是否合适，如果没达到条件则向下传递，向谁传递之前 set 进去。
### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Handler h1 = new ConcreteHandler1();
            Handler h2 = new ConcreteHandler2();
            Handler h3 = new ConcreteHandler3();
            h1.SetSuccessor(h2);
            h2.SetSuccessor(h3);

            int[] requests = { 2, 5, 14, 22, 18, 3, 27, 20 };

            foreach (int request in requests)
            {
                h1.HandleRequest(request);
            }

            Console.Read();

        }
    }

    abstract class Handler
    {
        protected Handler successor;

        public void SetSuccessor(Handler successor)
        {
            this.successor = successor;
        }

        public abstract void HandleRequest(int request);
    }

    class ConcreteHandler1 : Handler
    {
        public override void HandleRequest(int request)
        {
            if (request >= 0 && request < 10)
            {
                Console.WriteLine("{0}  处理请求  {1}",
                  this.GetType().Name, request);
            }
            else if (successor != null)
            {
                successor.HandleRequest(request);
            }
        }
    }

    class ConcreteHandler2 : Handler
    {
        public override void HandleRequest(int request)
        {
            if (request >= 10 && request < 20)
            {
                Console.WriteLine("{0}  处理请求  {1}",
                  this.GetType().Name, request);
            }
            else if (successor != null)
            {
                successor.HandleRequest(request);
            }
        }
    }

    class ConcreteHandler3 : Handler
    {
        public override void HandleRequest(int request)
        {
            if (request >= 20 && request < 30)
            {
                Console.WriteLine("{0}  处理请求  {1}",
                  this.GetType().Name, request);
            }
            else if (successor != null)
            {
                successor.HandleRequest(request);
            }
        }
    }

{% endhighlight %}

### 示例
不同的手机品牌运行软件

#### 步骤1 创建抽象处理类

{% highlight java %}

    //管理者
    abstract class Manager
    {
        protected string name;
        //管理者的上级
        protected Manager superior;

        public Manager(string name)
        {
            this.name = name;
        }

        //设置管理者的上级
        public void SetSuperior(Manager superior)
        {
            this.superior = superior;
        }

        //申请请求
        abstract public void RequestApplications(Request request);
    }

{% endhighlight %}

#### 步骤2 实现抽象类的具体类

{% highlight java %}

    //经理
    class CommonManager : Manager
    {
        public CommonManager(string name)
            : base(name)
        { }
        public override void RequestApplications(Request request)
        {

            if (request.RequestType == "请假" && request.Number <= 2)
            {
                Console.WriteLine("{0}:{1} 数量{2} 被批准", name, request.RequestContent, request.Number);
            }
            else
            {
                if (superior != null)
                    superior.RequestApplications(request);
            }

        }
    }

    //总监
    class Majordomo : Manager
    {
        public Majordomo(string name)
            : base(name)
        { }
        public override void RequestApplications(Request request)
        {

            if (request.RequestType == "请假" && request.Number <= 5)
            {
                Console.WriteLine("{0}:{1} 数量{2} 被批准", name, request.RequestContent, request.Number);
            }
            else
            {
                if (superior != null)
                    superior.RequestApplications(request);
            }

        }
    }

    //总经理
    class GeneralManager : Manager
    {
        public GeneralManager(string name)
            : base(name)
        { }
        public override void RequestApplications(Request request)
        {

            if (request.RequestType == "请假")
            {
                Console.WriteLine("{0}:{1} 数量{2} 被批准", name, request.RequestContent, request.Number);
            }
            else if (request.RequestType == "加薪" && request.Number <= 500)
            {
                Console.WriteLine("{0}:{1} 数量{2} 被批准", name, request.RequestContent, request.Number);
            }
            else if (request.RequestType == "加薪" && request.Number > 500)
            {
                Console.WriteLine("{0}:{1} 数量{2} 再说吧", name, request.RequestContent, request.Number);
            }
        }
    }

{% endhighlight %}


#### 步骤3 定义需处理的类

{% highlight java %}

    //申请
    class Request
    {
        //申请类别
        private string requestType;
        public string RequestType
        {
            get { return requestType; }
            set { requestType = value; }
        }

        //申请内容
        private string requestContent;
        public string RequestContent
        {
            get { return requestContent; }
            set { requestContent = value; }
        }

        //数量
        private int number;
        public int Number
        {
            get { return number; }
            set { number = value; }
        }
    }

{% endhighlight %}

#### 步骤4 使用命令模式

{% highlight java %}

     static void Main(string[] args)
        {

            CommonManager jinli = new CommonManager("金利");
            Majordomo zongjian = new Majordomo("宗剑");
            GeneralManager zhongjingli = new GeneralManager("钟精励");
            jinli.SetSuperior(zongjian);
            zongjian.SetSuperior(zhongjingli);

            Request request = new Request();
            request.RequestType = "请假";
            request.RequestContent = "小菜请假";
            request.Number = 1;
            jinli.RequestApplications(request);

            Request request2 = new Request();
            request2.RequestType = "请假";
            request2.RequestContent = "小菜请假";
            request2.Number = 4;
            jinli.RequestApplications(request2);

            Request request3 = new Request();
            request3.RequestType = "加薪";
            request3.RequestContent = "小菜请求加薪";
            request3.Number = 500;
            jinli.RequestApplications(request3);

            Request request4 = new Request();
            request4.RequestType = "加薪";
            request4.RequestContent = "小菜请求加薪";
            request4.Number = 1000;
            jinli.RequestApplications(request4);

            Console.Read();

        }

{% endhighlight %}
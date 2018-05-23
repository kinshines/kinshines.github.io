---
layout: post
title: 命令模式
author: kinshines
date:   2018-05-23
categories: design-pattern
permalink: /archivers/design-pattern-command
---

<p class="lead">
命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。
调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。
</p>

### 意图
将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。
### 主要解决
在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。
### 何时使用
在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。
### 如何解决
通过调用者调用接受者执行命令，顺序：调用者→接受者→命令。
### 关键代码
定义三个角色：
1. received 真正的命令执行对象 
2. Command 
3. invoker 使用命令对象的入口

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Receiver r = new Receiver();
            Command c = new ConcreteCommand(r);
            Invoker i = new Invoker();

            // Set and execute command 
            i.SetCommand(c);
            i.ExecuteCommand();

            Console.Read();

        }
    }

    abstract class Command
    {
        protected Receiver receiver;

        public Command(Receiver receiver)
        {
            this.receiver = receiver;
        }

        abstract public void Execute();
    }

    class ConcreteCommand : Command
    {
        public ConcreteCommand(Receiver receiver)
            :
          base(receiver) { }

        public override void Execute()
        {
            receiver.Action();
        }
    }

    class Receiver
    {
        public void Action()
        {
            Console.WriteLine("执行请求！");
        }
    }

    class Invoker
    {
        private Command command;

        public void SetCommand(Command command)
        {
            this.command = command;
        }

        public void ExecuteCommand()
        {
            command.Execute();
        }
    }

{% endhighlight %}

### 示例
不同的手机品牌运行软件

#### 步骤1 创建抽象命令类

{% highlight java %}

    //抽象命令
    public abstract class Command
    {
        protected Barbecuer receiver;

        public Command(Barbecuer receiver)
        {
            this.receiver = receiver;
        }

        //执行命令
        abstract public void ExcuteCommand();
    }


     //烤肉串者
    public class Barbecuer
    {
        public void BakeMutton()
        {
            Console.WriteLine("烤羊肉串!");
        }

        public void BakeChickenWing()
        {
            Console.WriteLine("烤鸡翅!");
        }
    }

{% endhighlight %}

#### 步骤2 实现抽象类的具体类

{% highlight java %}

    //烤羊肉串命令
    class BakeMuttonCommand : Command
    {
        public BakeMuttonCommand(Barbecuer receiver)
            : base(receiver)
        { }

        public override void ExcuteCommand()
        {
            receiver.BakeMutton();
        }
    }

    //烤鸡翅命令
    class BakeChickenWingCommand : Command
    {
        public BakeChickenWingCommand(Barbecuer receiver)
            : base(receiver)
        { }

        public override void ExcuteCommand()
        {
            receiver.BakeChickenWing();
        }
    }

{% endhighlight %}


#### 步骤3 定义命令对象的入口Invoke

{% highlight java %}

     //服务员
    public class Waiter
    {
        private IList<Command> orders = new List<Command>();

        //设置订单
        public void SetOrder(Command command)
        {
            if (command.ToString() == "命令模式.BakeChickenWingCommand")
            {
                Console.WriteLine("服务员：鸡翅没有了，请点别的烧烤。");
            }
            else
            {
                orders.Add(command);
                Console.WriteLine("增加订单：" + command.ToString() + "  时间：" + DateTime.Now.ToString());
            }
        }

        //取消订单
        public void CancelOrder(Command command)
        {
            orders.Remove(command);
            Console.WriteLine("取消订单：" + command.ToString() + "  时间：" + DateTime.Now.ToString());
        }

        //通知全部执行
        public void Notify()
        {
            foreach (Command cmd in orders)
            {
                cmd.ExcuteCommand();
            }
        }
    }

{% endhighlight %}

#### 步骤4 使用命令模式

{% highlight java %}

     static void Main(string[] args)
        {
            //开店前的准备
            Barbecuer boy = new Barbecuer();
            Command bakeMuttonCommand1 = new BakeMuttonCommand(boy);
            Command bakeMuttonCommand2 = new BakeMuttonCommand(boy);
            Command bakeChickenWingCommand1 = new BakeChickenWingCommand(boy);
            Waiter girl = new Waiter();

            //开门营业 顾客点菜
            girl.SetOrder(bakeMuttonCommand1);
            girl.SetOrder(bakeMuttonCommand2);
            girl.SetOrder(bakeChickenWingCommand1);

            //点菜完闭，通知厨房
            girl.Notify();

            Console.Read();

        }

{% endhighlight %}
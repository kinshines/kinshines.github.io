---
layout: post
title: 访问者模式
author: kinshines
date:   2018-05-25
categories: design-pattern
permalink: /archivers/design-pattern-visitor
---

<p class="lead">
在访问者模式（Visitor Pattern）中，我们使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。这种类型的设计模式属于行为型模式。
<br/>
根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。
</p>

### 意图
主要将数据结构与数据操作分离。
### 主要解决
稳定的数据结构和易变的操作耦合问题。
### 何时使用
需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中。
### 如何解决
在被访问的类里面加一个对外提供接待访问者的接口。
### 关键代码
在数据基础类里面有一个方法接受访问者，将自身引用传入访问者。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            ObjectStructure o = new ObjectStructure();
            o.Attach(new ConcreteElementA());
            o.Attach(new ConcreteElementB());

            ConcreteVisitor1 v1 = new ConcreteVisitor1();
            ConcreteVisitor2 v2 = new ConcreteVisitor2();

            o.Accept(v1);
            o.Accept(v2);

            Console.Read();
        }
    }

    abstract class Visitor
    {
        public abstract void VisitConcreteElementA(ConcreteElementA concreteElementA);

        public abstract void VisitConcreteElementB(ConcreteElementB concreteElementB);
    }

    class ConcreteVisitor1 : Visitor
    {
        public override void VisitConcreteElementA(ConcreteElementA concreteElementA)
        {
            Console.WriteLine("{0}被{1}访问", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void VisitConcreteElementB(ConcreteElementB concreteElementB)
        {
            Console.WriteLine("{0}被{1}访问", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }

    class ConcreteVisitor2 : Visitor
    {
        public override void VisitConcreteElementA(ConcreteElementA concreteElementA)
        {
            Console.WriteLine("{0}被{1}访问", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void VisitConcreteElementB(ConcreteElementB concreteElementB)
        {
            Console.WriteLine("{0}被{1}访问", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }

    abstract class Element
    {
        public abstract void Accept(Visitor visitor);
    }

    class ConcreteElementA : Element
    {
        public override void Accept(Visitor visitor)
        {
            visitor.VisitConcreteElementA(this);
        }

        public void OperationA()
        { }
    }

    class ConcreteElementB : Element
    {
        public override void Accept(Visitor visitor)
        {
            visitor.VisitConcreteElementB(this);
        }

        public void OperationB()
        { }
    }

    class ObjectStructure
    {
        private IList<Element> elements = new List<Element>();

        public void Attach(Element element)
        {
            elements.Add(element);
        }

        public void Detach(Element element)
        {
            elements.Remove(element);
        }

        public void Accept(Visitor visitor)
        {
            foreach (Element e in elements)
            {
                e.Accept(visitor);
            }
        }
    }

{% endhighlight %}

### 示例
男人女人在不同情境下的反应

#### 步骤1 创建一个表示元素的接口

{% highlight java %}

     //人
    abstract class Person
    {
        //接受
        public abstract void Accept(Action visitor);
    }

{% endhighlight %}

#### 步骤2 创建扩展了上述类的实体类

{% highlight java %}

    //男人
    class Man : Person
    {
        public override void Accept(Action visitor)
        {
            visitor.GetManConclusion(this);
        }
    }

    //女人
    class Woman : Person
    {
        public override void Accept(Action visitor)
        {
            visitor.GetWomanConclusion(this);
        }
    }

        //对象结构
    class ObjectStructure
    {
        private IList<Person> elements = new List<Person>();

        //增加
        public void Attach(Person element)
        {
            elements.Add(element);
        }
        //移除
        public void Detach(Person element)
        {
            elements.Remove(element);
        }
        //查看显示
        public void Display(Action visitor)
        {
            foreach (Person e in elements)
            {
                e.Accept(visitor);
            }
        }
    }

{% endhighlight %}


#### 步骤3 定义一个表示访问者的接口

{% highlight java %}

        //状态
    abstract class Action
    {
        //得到男人结论或反应
        public abstract void GetManConclusion(Man concreteElementA);
        //得到女人结论或反应
        public abstract void GetWomanConclusion(Woman concreteElementB);
    }

{% endhighlight %}

#### 步骤4 创建实现了上述类的实体访问者

{% highlight java %}

       //成功
    class Success : Action
    {
        public override void GetManConclusion(Man concreteElementA)
        {
            Console.WriteLine("{0}{1}时，背后多半有一个伟大的女人。", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void GetWomanConclusion(Woman concreteElementB)
        {
            Console.WriteLine("{0}{1}时，背后大多有一个不成功的男人。", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }
    //失败
    class Failing : Action
    {
        public override void GetManConclusion(Man concreteElementA)
        {
            Console.WriteLine("{0}{1}时，闷头喝酒，谁也不用劝。", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void GetWomanConclusion(Woman concreteElementB)
        {
            Console.WriteLine("{0}{1}时，眼泪汪汪，谁也劝不了。", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }
    //恋爱
    class Amativeness : Action
    {
        public override void GetManConclusion(Man concreteElementA)
        {
            Console.WriteLine("{0}{1}时，凡事不懂也要装懂。", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void GetWomanConclusion(Woman concreteElementB)
        {
            Console.WriteLine("{0}{1}时，遇事懂也装作不懂", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }
    //结婚
    class Marriage : Action
    {
        public override void GetManConclusion(Man concreteElementA)
        {
            Console.WriteLine("{0}{1}时，感慨道：恋爱游戏终结时，‘有妻徒刑’遥无期。", concreteElementA.GetType().Name, this.GetType().Name);
        }

        public override void GetWomanConclusion(Woman concreteElementB)
        {
            Console.WriteLine("{0}{1}时，欣慰曰：爱情长跑路漫漫，婚姻保险保平安。", concreteElementB.GetType().Name, this.GetType().Name);
        }
    }

{% endhighlight %}

#### 步骤5 使用访问者访问对象

{% highlight java %}

        static void Main(string[] args)
        {
            ObjectStructure o = new ObjectStructure();
            o.Attach(new Man());
            o.Attach(new Woman());

            Success v1 = new Success();
            o.Display(v1);

            Failing v2 = new Failing();
            o.Display(v2);

            Amativeness v3 = new Amativeness();
            o.Display(v3);

            Marriage v4 = new Marriage();
            o.Display(v4);

            Console.Read();
        }

{% endhighlight %}

### 后记
访问者模式适应于数据结构相对稳定的系统。

她把数据结构和作用域结构上的操作之间的耦合解脱开，使得操作集合可以相对自由地演化，目的是要把处理从数据结构分离出来。

有比较稳定的数据结构，又有易于变化的算法，使用访问者模式就是比较合适的，因为访问者模式使得算法操作的增加变得容易，缺点也是使增加新的数据结构变得困难了。

大多数时候你并不要访问者模式，但当你一旦需要访问者模式时，那就是真的需要它了。

访问者模式的能力和复杂性是把双刃剑，只有当你真正需要它的时候，才考虑使用它。
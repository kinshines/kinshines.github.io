---
layout: post
title: 建造者模式
author: kinshines
date:   2018-05-19
categories: design-pattern
permalink: /archivers/design-pattern-builder
---

<p class="lead">
建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
<br/>
一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。
</p>

### 意图
将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。
### 主要解决
主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。
### 何时使用
一些基本部件不会变，而其组合经常变化的时候。
### 如何解决
将变与不变分离开。
### 关键代码
建造者：创建和提供实例，导演：管理建造出来的实例的依赖关系。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Director director = new Director();
            Builder b1 = new ConcreteBuilder1();
            Builder b2 = new ConcreteBuilder2();

            director.Construct(b1);
            Product p1 = b1.GetResult();
            p1.Show();

            director.Construct(b2);
            Product p2 = b2.GetResult();
            p2.Show();

            Console.Read();
        }
    }

    class Director
    {
        public void Construct(Builder builder)
        {
            builder.BuildPartA();
            builder.BuildPartB();
        }
    }

    abstract class Builder
    {
        public abstract void BuildPartA();
        public abstract void BuildPartB();
        public abstract Product GetResult();
    }

    class ConcreteBuilder1 : Builder
    {
        private Product product = new Product();

        public override void BuildPartA()
        {
            product.Add("部件A");
        }

        public override void BuildPartB()
        {
            product.Add("部件B");
        }

        public override Product GetResult()
        {
            return product;
        }
    }

    class ConcreteBuilder2 : Builder
    {
        private Product product = new Product();
        public override void BuildPartA()
        {
            product.Add("部件X");
        }

        public override void BuildPartB()
        {
            product.Add("部件Y");
        }

        public override Product GetResult()
        {
            return product;
        }
    }

    class Product
    {
        IList<string> parts = new List<string>();

        public void Add(string part)
        {
            parts.Add(part);
        }

        public void Show()
        {
            Console.WriteLine("\n产品 创建 ----");
            foreach (string part in parts)
            {
                Console.WriteLine(part);
            }
        }
    }

{% endhighlight %}

### 示例
WinForm中绘画出胖子和瘦子

#### 步骤1 创建建造者抽象类

{% highlight java %}

    abstract class PersonBuilder
    {
        protected Graphics g;
        protected Pen p;

        public PersonBuilder(Graphics g, Pen p)
        {
            this.g = g;
            this.p = p;
        }

        public abstract void BuildHead();
        public abstract void BuildBody();
        public abstract void BuildArmLeft();
        public abstract void BuildArmRight();
        public abstract void BuildLegLeft();
        public abstract void BuildLegRight();
    }

{% endhighlight %}

#### 步骤2 实现建造者类。

{% highlight java %}

    class PersonThinBuilder : PersonBuilder
    {
        public PersonThinBuilder(Graphics g, Pen p)
            : base(g, p)
        { }

        public override void BuildHead()
        {
            g.DrawEllipse(p, 50, 20, 30, 30);
        }

        public override void BuildBody()
        {
            g.DrawRectangle(p, 60, 50, 10, 50);
        }

        public override void BuildArmLeft()
        {
            g.DrawLine(p, 60, 50, 40, 100);
        }

        public override void BuildArmRight()
        {
            g.DrawLine(p, 70, 50, 90, 100);
        }

        public override void BuildLegLeft()
        {
            g.DrawLine(p, 60, 100, 45, 150);
        }

        public override void BuildLegRight()
        {
            g.DrawLine(p, 70, 100, 85, 150);
        }
    }

    class PersonFatBuilder : PersonBuilder
    {
        public PersonFatBuilder(Graphics g, Pen p)
            : base(g, p)
        { }

        public override void BuildHead()
        {
            g.DrawEllipse(p, 50, 20, 30, 30);
        }

        public override void BuildBody()
        {
            g.DrawEllipse(p, 45, 50, 40, 50);
        }

        public override void BuildArmLeft()
        {
            g.DrawLine(p, 50, 50, 30, 100);
        }

        public override void BuildArmRight()
        {
            g.DrawLine(p, 80, 50, 100, 100);
        }

        public override void BuildLegLeft()
        {
            g.DrawLine(p, 60, 100, 45, 150);
        }

        public override void BuildLegRight()
        {
            g.DrawLine(p, 70, 100, 85, 150);
        }
    }

{% endhighlight %}

#### 步骤3 创建指挥者类。

{% highlight java %}

    class PersonDirector
    {
        private PersonBuilder pb;
        public PersonDirector(string type, Graphics g, Pen p)
        {
            string assemblyName="建造者模式";
            object[] args = new object[2];
            args[0] = g;
            args[1] = p;

            this.pb = (PersonBuilder)Assembly.Load(assemblyName).CreateInstance(assemblyName+".Person" + type + "Builder", false, BindingFlags.Default, null, args, null, null);
        }

        public void CreatePerson()
        {
            pb.BuildHead();
            pb.BuildBody();
            pb.BuildArmLeft();
            pb.BuildArmRight();
            pb.BuildLegLeft();
            pb.BuildLegRight();
        }
    }

{% endhighlight %}

#### 步骤4 调用指挥者创建具体的产品。

{% highlight java %}

    Pen p = new Pen(Color.Yellow);
    PersonDirector pdThin = new PersonDirector("Thin",pictureBox1.CreateGraphics(),p);
    pdThin.CreatePerson();

    PersonDirector pdFat = new PersonDirector("Fat", pictureBox2.CreateGraphics(), p);
    pdFat.CreatePerson();

{% endhighlight %}
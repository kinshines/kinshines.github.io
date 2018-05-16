---
layout: post
title: 代理模式
author: kinshines
date:   2018-05-16
categories: design-pattern
permalink: /archivers/design-pattern-prototype
---

<p class="lead">原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
<br/>
这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。
</p>

### 意图
用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
### 主要解决
在运行期建立和删除原型。
### 何时使用
1. 当一个系统应该独立于它的产品创建，构成和表示时。 
2. 当要实例化的类是在运行时刻指定时，例如，通过动态装载。
3. 为了避免创建一个与产品类层次平行的工厂类层次时。 
4. 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。
### 如何解决
利用已有的一个原型对象，快速地生成和原型对象一样的实例。
### 关键代码
 1. 实现克隆操作，在 JAVA 继承 Cloneable，重写 clone()，在 .NET 中可以使用 Object 类的 MemberwiseClone() 方法来实现对象的浅拷贝或通过序列化的方式来实现深拷贝。
 2. 原型模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系，它同样要求这些"易变类"拥有稳定的接口。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            ConcretePrototype1 p1 = new ConcretePrototype1("I");
            ConcretePrototype1 c1 = (ConcretePrototype1)p1.Clone();
            Console.WriteLine("Cloned: {0}", c1.Id);

            ConcretePrototype2 p2 = new ConcretePrototype2("II");
            ConcretePrototype2 c2 = (ConcretePrototype2)p2.Clone();
            Console.WriteLine("Cloned: {0}", c2.Id);

            // Wait for user 
            Console.Read();

        }
    }

    abstract class Prototype
    {
        private string id;

        // Constructor 
        public Prototype(string id)
        {
            this.id = id;
        }

        // Property 
        public string Id
        {
            get { return id; }
        }

        public abstract Prototype Clone();
    }

    class ConcretePrototype1 : Prototype
    {
        // Constructor 
        public ConcretePrototype1(string id)
            : base(id)
        {
        }

        public override Prototype Clone()
        {
            // Shallow copy 
            return (Prototype)this.MemberwiseClone();
        }
    }


    class ConcretePrototype2 : Prototype
    {
        // Constructor 
        public ConcretePrototype2(string id)
            : base(id)
        {
        }

        public override Prototype Clone()
        {
            // Shallow copy 
            return (Prototype)this.MemberwiseClone();
        }
    }

{% endhighlight %}

### 示例
复制简历

#### 步骤1 创建一个实现ICloneable的类

{% highlight java %}

    //简历
    class Resume : ICloneable
    {
        private string name;
        private string sex;
        private string age;

        private WorkExperience work;

        public Resume(string name)
        {
            this.name = name;
            work = new WorkExperience();
        }

        private Resume(WorkExperience work)
        {
            this.work = (WorkExperience)work.Clone();
        }

        //设置个人信息
        public void SetPersonalInfo(string sex, string age)
        {
            this.sex = sex;
            this.age = age;
        }
        //设置工作经历
        public void SetWorkExperience(string workDate, string company)
        {
            work.WorkDate = workDate;
            work.Company = company;
        }

        //显示
        public void Display()
        {
            Console.WriteLine("{0} {1} {2}", name, sex, age);
            Console.WriteLine("工作经历：{0} {1}", work.WorkDate, work.Company);
        }

        public Object Clone()
        {
            Resume obj = new Resume(this.work);

            obj.name = this.name;
            obj.sex = this.sex;
            obj.age = this.age;
            return obj;
        }
    }

    //工作经历
    class WorkExperience : ICloneable
    {
        private string workDate;
        public string WorkDate
        {
            get { return workDate; }
            set { workDate = value; }
        }
        private string company;
        public string Company
        {
            get { return company; }
            set { company = value; }
        }

        public Object Clone()
        {
            return (Object)this.MemberwiseClone();
        }
    }

{% endhighlight %}

#### 步骤2 实现复制

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Resume a = new Resume("大鸟");
            a.SetPersonalInfo("男", "29");
            a.SetWorkExperience("1998-2000", "XX公司");

            Resume b = (Resume)a.Clone();
            b.SetWorkExperience("1998-2006", "YY企业");

            Resume c = (Resume)a.Clone();
            c.SetWorkExperience("1998-2003", "ZZ企业");

            a.Display();
            b.Display();
            c.Display();

            Console.Read();

        }
    }

{% endhighlight %}
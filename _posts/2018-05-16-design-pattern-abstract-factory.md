---
layout: post
title: 抽象工厂模式
author: kinshines
date:   2018-05-16
categories: design-pattern
permalink: /archivers/design-pattern-abstract-factory
---

<p class="lead">抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
<br/>
在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。
</p>

### 意图
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
### 主要解决
主要解决接口选择的问题。
### 何时使用
系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。
### 如何解决
在一个产品族里面，定义多个产品。
### 关键代码
在一个工厂里聚合多个同类产品。
### 与工厂模式的不同之处
抽象工厂的更大意义在于，使用抽象工厂可以在一个产品族里面，定义多个产品，在一个工厂里聚合多个同类产品。

### 示例
使用抽象工厂实现数据库之间切换

#### 步骤1 创建产品接口

{% highlight java %}

    class User
    {
        private int _id;
        public int ID
        {
            get { return _id; }
            set { _id = value; }
        }

        private string _name;
        public string Name
        {
            get { return _name; }
            set { _name = value; }
        }
    }

    class Department
    {
        private int _id;
        public int ID
        {
            get { return _id; }
            set { _id = value; }
        }

        private string _deptName;
        public string DeptName
        {
            get { return _deptName; }
            set { _deptName = value; }
        }
    }

    interface IUser
    {
        void Insert(User user);

        User GetUser(int id);
    }

    interface IDepartment
    {
        void Insert(Department department);

        Department GetDepartment(int id);
    }

{% endhighlight %}

#### 步骤2 实现该产品接口的实体产品

{% highlight java %}

    class SqlserverUser : IUser
    {
        public void Insert(User user)
        {
            Console.WriteLine("在Sqlserver中给User表增加一条记录");
        }

        public User GetUser(int id)
        {
            Console.WriteLine("在Sqlserver中根据ID得到User表一条记录");
            return null;
        }
    }

    class AccessUser : IUser
    {
        public void Insert(User user)
        {
            Console.WriteLine("在Access中给User表增加一条记录");
        }

        public User GetUser(int id)
        {
            Console.WriteLine("在Access中根据ID得到User表一条记录");
            return null;
        }
    }

    class SqlserverDepartment : IDepartment
    {
        public void Insert(Department department)
        {
            Console.WriteLine("在Sqlserver中给Department表增加一条记录");
        }

        public Department GetDepartment(int id)
        {
            Console.WriteLine("在Sqlserver中根据ID得到Department表一条记录");
            return null;
        }
    }

    class AccessDepartment : IDepartment
    {
        public void Insert(Department department)
        {
            Console.WriteLine("在Access中给Department表增加一条记录");
        }

        public Department GetDepartment(int id)
        {
            Console.WriteLine("在Access中根据ID得到Department表一条记录");
            return null;
        }
    }

{% endhighlight %}

#### 步骤3 创建抽象工厂

{% highlight java %}

    interface IFactory
    {
        IUser CreateUser();

        IDepartment CreateDepartment();
    }

{% endhighlight %}

#### 步骤4 创建扩展了 IFactory 的工厂类，生成实体类的对象。

{% highlight java %}

    class SqlServerFactory : IFactory
    {
        public IUser CreateUser()
        {
            return new SqlserverUser();
        }

        public IDepartment CreateDepartment()
        {
            return new SqlserverDepartment();
        }
    }

    class AccessFactory : IFactory
    {
        public IUser CreateUser()
        {
            return new AccessUser();
        }

        public IDepartment CreateDepartment()
        {
            return new AccessDepartment();
        }
    }

{% endhighlight %}

#### 步骤5 使用实际工厂

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            IFactory operFactory = new AddFactory();
            Operation oper = operFactory.CreateOperation();
            oper.NumberA = 1;
            oper.NumberB = 2;
            double result=oper.GetResult();

            Console.WriteLine(result);

            Console.Read();
        }
    }

{% endhighlight %}
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
工厂方法模式，定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类
### 主要解决
主要解决接口选择的问题。
### 何时使用
系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。
### 如何解决
在一个产品族里面，定义多个产品。
### 关键代码
在一个工厂里聚合多个同类产品。
### 与简单工厂的不同之处
简单工厂模式的最大优点在于工厂类中包含了必要的逻辑判断，根据客户端的选择动态实例化相关的类，对于客户端来说，去除了与具体产品的依赖

抽象工厂方法模式实现时，客户端需要决定实例化哪一个工厂来实现运算类，选择判断的问题还是存在的，也就是说，抽象工厂方法把简单工厂的内部逻辑移到了客户端代码来进行。你想要加功能，本来是该工厂类，而现在修改客户端！

抽象工厂的更大意义在于，使用抽象工厂可以在一个产品族里面，定义多个产品，在一个工厂里聚合多个同类产品。

### 示例
使用抽象工厂做一个计算器

#### 步骤1 创建一个接口

{% highlight java %}

    /// <summary>
    /// 运算类
    /// </summary>
    class Operation
    {
        private double _numberA = 0;
        private double _numberB = 0;

        public double NumberA
        {
            get { return _numberA; }
            set { _numberA = value; }
        }

        public double NumberB
        {
            get { return _numberB; }
            set { _numberB = value; }
        }

        /// <summary>
        /// 得到运算结果
        /// </summary>
        /// <returns></returns>
        public virtual double GetResult()
        {
            double result = 0;
            return result;
        }
    }

{% endhighlight %}

#### 步骤2 实现该接口的实体类

{% highlight java %}

    /// <summary>
    /// 加法类
    /// </summary>
    class OperationAdd : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            result = NumberA + NumberB;
            return result;
        }
    }

    /// <summary>
    /// 减法类
    /// </summary>
    class OperationSub : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            result = NumberA - NumberB;
            return result;
        }
    }
    /// <summary>
    /// 乘法类
    /// </summary>
    class OperationMul : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            result = NumberA * NumberB;
            return result;
        }
    }
    /// <summary>
    /// 除法类
    /// </summary>
    class OperationDiv : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            if (NumberB == 0)
                throw new Exception("除数不能为0。");
            result = NumberA / NumberB;
            return result;
        }
    }

{% endhighlight %}

#### 步骤3 创建抽象类来获取工厂

{% highlight java %}

    /// <summary>
    /// 工厂方法
    /// </summary>
    interface IFactory
    {
        Operation CreateOperation();
    }

{% endhighlight %}

#### 步骤4 创建扩展了 IFactory 的工厂类，生成实体类的对象。

{% highlight java %}

    /// <summary>
    /// 专门负责生产“+”的工厂
    /// </summary>
    class AddFactory : IFactory
    {
        public Operation CreateOperation()
        {
            return new OperationAdd();
        }
    }

    /// <summary>
    /// 专门负责生产“-”的工厂
    /// </summary>
    class SubFactory : IFactory
    {
        public Operation CreateOperation()
        {
            return new OperationSub();
        }
    }

    /// <summary>
    /// 专门负责生产“*”的工厂
    /// </summary>
    class MulFactory : IFactory
    {
        public Operation CreateOperation()
        {
            return new OperationMul();
        }
    }

    /// <summary>
    /// 专门负责生产“/”的工厂
    /// </summary>
    class DivFactory : IFactory
    {
        public Operation CreateOperation()
        {
            return new OperationDiv();
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
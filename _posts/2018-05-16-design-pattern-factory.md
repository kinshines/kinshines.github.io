---
layout: post
title: 工厂模式
author: kinshines
date:   2018-05-16
categories: design-pattern
permalink: /archivers/design-pattern-factory
---

<p class="lead">工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
<br />
在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
</p>

### 意图
定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
### 主要解决
主要解决接口选择的问题。
### 何时使用
我们明确地计划不同条件下创建不同实例时。
### 如何解决
让其子类实现工厂接口，返回的也是一个抽象的产品。
### 关键代码
创建过程在其子类执行。
### 与简单工厂的不同之处
简单工厂模式的最大优点在于工厂类中包含了必要的逻辑判断，根据客户端的选择动态实例化相关的类，对于客户端来说，去除了与具体产品的依赖

### 示例
使用工厂方法做一个计算器

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
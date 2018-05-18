---
layout: post
title: 外观模式
author: kinshines
date:   2018-05-18
categories: design-pattern
permalink: /archivers/design-pattern-facade
---

<p class="lead">外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。
<br/>
这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。
</p>

### 意图
为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。
### 主要解决
降低访问复杂系统的内部子系统时的复杂度，简化客户端与之的接口。
### 何时使用
1. 客户端不需要知道系统内部的复杂联系，整个系统只需提供一个"接待员"即可。 
2. 定义系统的入口。

### 如何解决
客户端不与系统耦合，外观类与系统耦合。
### 关键代码
在客户端和复杂系统之间再加一层，这一层将调用顺序、依赖关系等处理好。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Facade facade = new Facade();

            facade.MethodA();
            facade.MethodB();

            Console.Read();

        }
    }

    class SubSystemOne
    {
        public void MethodOne()
        {
            Console.WriteLine(" 子系统方法一");
        }
    }

    class SubSystemTwo
    {
        public void MethodTwo()
        {
            Console.WriteLine(" 子系统方法二");
        }
    }

    class SubSystemThree
    {
        public void MethodThree()
        {
            Console.WriteLine(" 子系统方法三");
        }
    }

    class SubSystemFour
    {
        public void MethodFour()
        {
            Console.WriteLine(" 子系统方法四");
        }
    }

    class Facade
    {
        SubSystemOne one;
        SubSystemTwo two;
        SubSystemThree three;
        SubSystemFour four;

        public Facade()
        {
            one = new SubSystemOne();
            two = new SubSystemTwo();
            three = new SubSystemThree();
            four = new SubSystemFour();
        }

        public void MethodA()
        {
            Console.WriteLine("\n方法组A() ---- ");
            one.MethodOne();
            two.MethodTwo();
            four.MethodFour();
        }

        public void MethodB()
        {
            Console.WriteLine("\n方法组B() ---- ");
            two.MethodTwo();
            three.MethodThree();
        }
    }

{% endhighlight %}

### 示例
股票和基金的关系

#### 步骤1 创建一系列的子系统

{% highlight java %}

    //股票1
    class Stock1
    {
        //卖股票
        public void Sell()
        {
            Console.WriteLine(" 股票1卖出");
        }

        //买股票
        public void Buy()
        {
            Console.WriteLine(" 股票1买入");
        }
    }

    //股票2
    class Stock2
    {
        //卖股票
        public void Sell()
        {
            Console.WriteLine(" 股票2卖出");
        }

        //买股票
        public void Buy()
        {
            Console.WriteLine(" 股票2买入");
        }
    }

    //股票3
    class Stock3
    {
        //卖股票
        public void Sell()
        {
            Console.WriteLine(" 股票3卖出");
        }

        //买股票
        public void Buy()
        {
            Console.WriteLine(" 股票3买入");
        }
    }

    //国债1
    class NationalDebt1
    {
        //卖国债
        public void Sell()
        {
            Console.WriteLine(" 国债1卖出");
        }

        //买国债
        public void Buy()
        {
            Console.WriteLine(" 国债1买入");
        }
    }

    //房地产1
    class Realty1
    {
        //卖房地产
        public void Sell()
        {
            Console.WriteLine(" 房产1卖出");
        }

        //买房地产
        public void Buy()
        {
            Console.WriteLine(" 房产1买入");
        }
    }

{% endhighlight %}

#### 步骤2 创建外观类。

{% highlight java %}

    class Fund
    {
        Stock1 gu1;
        Stock2 gu2;
        Stock3 gu3;
        NationalDebt1 nd1;
        Realty1 rt1;

        public Fund()
        {
            gu1 = new Stock1();
            gu2 = new Stock2();
            gu3 = new Stock3();
            nd1 = new NationalDebt1();
            rt1 = new Realty1();
        }

        public void BuyFund()
        {
            gu1.Buy();
            gu2.Buy();
            gu3.Buy();
            nd1.Buy();
            rt1.Buy();
        }

        public void SellFund()
        {
            gu1.Sell();
            gu2.Sell();
            gu3.Sell();
            nd1.Sell();
            rt1.Sell();
        }

    }

{% endhighlight %}

#### 步骤3 调用外观。

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {

            Fund jijin = new Fund();

            jijin.BuyFund();
            jijin.SellFund();

            Console.Read();
        }
    }

{% endhighlight %}
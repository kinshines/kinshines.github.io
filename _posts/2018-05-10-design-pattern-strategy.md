---
layout: post
title: 策略模式
author: kinshines
date:   2018-05-10
categories: design-pattern
permalink: /archivers/design-pattern-strategy
---

<p class="lead">在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。
<br/>
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。</p>

### 意图
定义了算法家族，分别封装起来，让它们之间可以相互替换，此模式让算法的变化，不会影响到使用算法的用户
### 主要解决
在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。
### 何时使用
一个系统有许多许多类，而区分它们的只是他们直接的行为。
### 如何解决
将这些算法封装成一个一个的类，任意地替换。
### 关键代码
实现同一个接口。
### 示例
商场收银软件，营业员根据客户所购买的商品的单价和数量，通过优惠算法，向客户收费

#### 步骤1 创建一个接口

{% highlight java %}

    abstract class CashSuper
    {
        public abstract double acceptCash(double money);
    }

{% endhighlight %}

#### 步骤2 创建实现接口的实体类

{% highlight java %}

    /// <summary>
    /// 原价
    /// </summary>
    class CashNormal : CashSuper
    {
        public override double acceptCash(double money)
        {
            return money;
        } 
    }

    /// <summary>
    /// 打折
    /// </summary>
    class CashRebate : CashSuper
    {
        private double moneyRebate = 1d;
        public CashRebate(string moneyRebate)
        {
            this.moneyRebate = double.Parse(moneyRebate);
        }

        public override double acceptCash(double money)
        {
            return money * moneyRebate;
        } 
    }

    /// <summary>
    /// 满减
    /// </summary>
    class CashReturn : CashSuper
    {
        private double moneyCondition = 0.0d;
        private double moneyReturn = 0.0d;
        
        public CashReturn(string moneyCondition,string moneyReturn)
        {
            this.moneyCondition = double.Parse(moneyCondition);
            this.moneyReturn = double.Parse(moneyReturn);
        }

        public override double acceptCash(double money)
        {
            double result = money;
            if (money >= moneyCondition)
                result=money- Math.Floor(money / moneyCondition) * moneyReturn;
                
            return result;
        } 
    }

{% endhighlight %}

#### 步骤3 创建 Context 类

{% highlight java %}

    //收费策略Context
class CashContext
{
    //声明一个现金收费父类对象
    private CashSuper cs;

    //设置策略行为，参数为具体的现金收费子类（正常，打折或返利）
    public CashContext(CashSuper csuper)
    {
        this.cs = csuper;
    }

    //得到现金促销计算结果（利用了多态机制，不同的策略行为导致不同的结果）
    public double GetResult(double money)
    {
        return cs.acceptCash(money);
    }
}

{% endhighlight %}

#### 步骤4 使用 Context 来查看当它改变策略 Strategy 时的行为变化。

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            double total = 0.0d;//用于总计
            CashContext cc = new CashContext(new CashNormal());
            total=total+cc.GetResult(100);
            cc=new CashContext(new CashReturn("300", "100"));
            total=total+cc.GetResult(300);
            cc=new CashContext(new CashRebate("0.8"));
            total=total+cc.GetResult(200);
        }
    }

{% endhighlight %}

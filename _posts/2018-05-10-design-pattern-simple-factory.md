---
layout: post
title: 简单工厂模式
author: kinshines
date:   2018-05-10
categories: design-pattern
permalink: /archivers/design-pattern-simple-factory
---

<p class="lead">工厂模式（Factory Pattern）是最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。<br/>
在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。</p>

### 意图
定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行
### 主要解决
主要解决接口选择的问题
### 何时使用
我们明确地计划不同条件下创建不同实例时。
### 如何解决
让其子类实现工厂接口，返回的是一个抽象的产品。
### 关键代码
创建过程在其子类执行。
### 示例
设计一个计算器控制台程序，要求输入两个数和运算符号，得到结果

#### 步骤1 创建一个接口

{% highlight java %}

    /// <summary>
    /// 运算类
    /// </summary>
    public class Operation
    {
        private double _numberA = 0;
        private double _numberB = 0;

        /// <summary>
        /// 数字A
        /// </summary>
        public double NumberA
        {
            get
            {
                return _numberA;
            }
            set
            {
                _numberA = value;
            }
        }

        /// <summary>
        /// 数字B
        /// </summary>
        public double NumberB
        {
            get
            {
                return _numberB;
            }
            set
            {
                _numberB = value;
            }
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

        /// <summary>
        /// 检查输入的字符串是否准确
        /// </summary>
        /// <param name="currentNumber"></param>
        /// <param name="inputString"></param>
        /// <returns></returns>
        public static string checkNumberInput(string currentNumber, string inputString)
        {
            string result = "";
            if (inputString == ".")
            {
                if (currentNumber.IndexOf(".") < 0)
                {
                    if (currentNumber.Length == 0)
                        result = "0" + inputString;
                    else
                        result = currentNumber + inputString;
                }
            }
            else if (currentNumber == "0")
            {
                result = inputString;
            }
            else
            {
                result = currentNumber + inputString;
            }

            return result;
        }
    }

{% endhighlight %}

#### 步骤2 创建实现接口的实体类

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

    /// <summary>
    /// 平方类
    /// </summary>
    class OperationSqr : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            result = NumberB * NumberB;
            return result;
        }
    }

    /// <summary>
    /// 平方根类
    /// </summary>
    class OperationSqrt : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            if (NumberB < 0)
                throw new Exception("负数不能开平方根。");
            result = Math.Sqrt(NumberB);
            return result;
        }
    }

    /// <summary>
    /// 相反数类
    /// </summary>
    class OperationReverse : Operation
    {
        public override double GetResult()
        {
            double result = 0;
            result = -NumberB;
            return result;
        }
    }

{% endhighlight %}

#### 步骤3 创建一个工厂，生成基于给定信息的实体类的对象

{% highlight java %}

    /// <summary>
    /// 运算类工厂
    /// </summary>
public class OperationFactory
{
    public static Operation createOperate(string operate)
    {
        Operation oper = null;
        switch (operate)
        {
            case "+":
                {
                    oper = new OperationAdd();
                    break;
                }
            case "-":
                {
                    oper = new OperationSub();
                    break;
                }
            case "*":
                {
                    oper = new OperationMul();
                    break;
                }
            case "/":
                {
                    oper = new OperationDiv();
                    break;
                }
            case "sqr":
                {
                    oper = new OperationSqr();
                    break;
                }
            case "sqrt":
                {
                    oper = new OperationSqrt();
                    break;
                }
            case "+/-":
                {
                    oper = new OperationReverse();
                    break;
                }
        }

        return oper;
    }
}

{% endhighlight %}

#### 步骤4 使用该工厂，通过传递类型信息来获取实体类的对象

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Console.Write("请输入数字A：");
                string strNumberA = Console.ReadLine();
                Console.Write("请选择运算符号(+、-、*、/)：");
                string strOperate = Console.ReadLine();
                Console.Write("请输入数字B：");
                string strNumberB = Console.ReadLine();
                string strResult = "";

                Operation oper;
                oper = OperationFactory.createOperate(strOperate);
                oper.NumberA = Convert.ToDouble(strNumberA);
                oper.NumberB = Convert.ToDouble(strNumberB);
                strResult = oper.GetResult().ToString();

                Console.WriteLine("结果是：" + strResult);

                Console.ReadLine();


            }
            catch (Exception ex)
            {
                Console.WriteLine("您的输入有错：" + ex.Message);
            }
        }
    }

{% endhighlight %}

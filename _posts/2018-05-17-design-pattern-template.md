---
layout: post
title: 模板模式
author: kinshines
date:   2018-05-17
categories: design-pattern
permalink: /archivers/design-pattern-template
---

<p class="lead">在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。
<br/>
这种类型的设计模式属于行为型模式。
</p>

### 意图
定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。
### 主要解决
一些方法通用，却在每一个子类都重新写了这一方法。
### 何时使用
有一些通用的方法。
### 如何解决
将这些通用算法抽象出来。
### 关键代码
在抽象类实现，其他步骤在子类实现。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            AbstractClass c;

            c = new ConcreteClassA();
            c.TemplateMethod();

            c = new ConcreteClassB();
            c.TemplateMethod();

            Console.Read();

        }
    }

    abstract class AbstractClass
    {
        public abstract void PrimitiveOperation1();
        public abstract void PrimitiveOperation2();

        public void TemplateMethod()
        {
            PrimitiveOperation1();
            PrimitiveOperation2();
            Console.WriteLine("");
        }
    }

    class ConcreteClassA : AbstractClass
    {
        public override void PrimitiveOperation1()
        {
            Console.WriteLine("具体类A方法1实现");
        }
        public override void PrimitiveOperation2()
        {
            Console.WriteLine("具体类A方法2实现");
        }
    }

    class ConcreteClassB : AbstractClass
    {
        public override void PrimitiveOperation1()
        {
            Console.WriteLine("具体类B方法1实现");
        }
        public override void PrimitiveOperation2()
        {
            Console.WriteLine("具体类B方法2实现");
        }
    }

{% endhighlight %}

### 示例
同样一份试题，不同的人给出不同的答案

#### 步骤1 创建一个抽象类，它的模板方法被设置为 final(Java)或sealed override(C#)。

{% highlight java %}

        public void TestQuestion2()
        {
            Console.WriteLine(" 杨过、程英、陆无双铲除了情花，造成[ ] a.使这种植物不再害人 b.使一种珍稀物种灭绝 c.破坏了那个生物圈的生态平衡 d.造成该地区沙漠化  ");
            Console.WriteLine("答案：" + Answer2());
        }

        public void TestQuestion3()
        {
            Console.WriteLine(" 蓝凤凰的致使华山师徒、桃谷六仙呕吐不止,如果你是大夫,会给他们开什么药[ ] a.阿司匹林 b.牛黄解毒片 c.氟哌酸 d.让他们喝大量的生牛奶 e.以上全不对   ");
            Console.WriteLine("答案：" + Answer3());
        }

        protected virtual string Answer1()
        {
            return "";
        }

        protected virtual string Answer2()
        {
            return "";
        }

        protected virtual string Answer3()
        {
            return "";
        }
    }

{% endhighlight %}

#### 步骤2 创建扩展了上述类的实体类。

{% highlight java %}

    //学生甲抄的试卷
    class TestPaperA : TestPaper
    {
        protected override string Answer1()
        {
            return "b";
        }

        protected override string Answer2()
        {
            return "c";
        }

        protected override string Answer3()
        {
            return "a";
        }
    }
    //学生乙抄的试卷
    class TestPaperB : TestPaper
    {
        protected override string Answer1()
        {
            return "c";
        }

        protected override string Answer2()
        {
            return "a";
        }

        protected override string Answer3()
        {
            return "a";
        }
    }

{% endhighlight %}

#### 步骤3 调用模板方法。

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("学生甲抄的试卷：");
            TestPaper studentA = new TestPaperA();
            studentA.TestQuestion1();
            studentA.TestQuestion2();
            studentA.TestQuestion3();

            Console.WriteLine("学生乙抄的试卷：");
            TestPaper studentB = new TestPaperB();
            studentB.TestQuestion1();
            studentB.TestQuestion2();
            studentB.TestQuestion3();

            Console.Read();
        }
    }

{% endhighlight %}
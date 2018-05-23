---
layout: post
title: 迭代器模式
author: kinshines
date:   2018-05-23
categories: design-pattern
permalink: /archivers/design-pattern-iterator
---

<p class="lead">
迭代器模式（Iterator Pattern）是 Java 和 .Net 编程环境中非常常用的设计模式。这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。
<br/>
迭代器模式属于行为型模式。
</p>

### 意图
提供一种方法顺序访问一个聚合对象中各个元素, 而又无须暴露该对象的内部表示。
### 主要解决
不同的方式来遍历整个整合对象。
### 何时使用
遍历一个聚合对象。
### 如何解决
把在元素之间游走的责任交给迭代器，而不是聚合对象。
### 关键代码
定义接口：hasNext, next。

### 基本代码

{% highlight java %}

    abstract class Aggregate
    {
        public abstract Iterator CreateIterator();
    }

    class ConcreteAggregate : Aggregate
    {
        private IList<object> items = new List<object>();
        public override Iterator CreateIterator()
        {
            return new ConcreteIterator(this);
        }

        public int Count
        {
            get { return items.Count; }
        }

        public object this[int index]
        {
            get { return items[index]; }
            set { items.Insert(index, value); }
        }
    }

    abstract class Iterator
    {
        public abstract object First();
        public abstract object Next();
        public abstract bool IsDone();
        public abstract object CurrentItem();
    }

    class ConcreteIterator : Iterator
    {
        private ConcreteAggregate aggregate;
        private int current = 0;

        public ConcreteIterator(ConcreteAggregate aggregate)
        {
            this.aggregate = aggregate;
        }

        public override object First()
        {
            return aggregate[0];
        }

        public override object Next()
        {
            object ret = null;
            current++;

            if (current < aggregate.Count)
            {
                ret = aggregate[current];
            }

            return ret;
        }

        public override object CurrentItem()
        {
            return aggregate[current];
        }

        public override bool IsDone()
        {
            return current >= aggregate.Count ? true : false;
        }

    }

    class Program
    {
        static void Main(string[] args)
        {
            ConcreteAggregate a = new ConcreteAggregate();

            a[0] = "大鸟";
            a[1] = "小菜";
            a[2] = "行李";
            a[3] = "老外";
            a[4] = "公交内部员工";
            a[5] = "小偷";

            Iterator i = new ConcreteIterator(a);
            //Iterator i = new ConcreteIteratorDesc(a);
            object item = i.First();
            while (!i.IsDone())
            {
                Console.WriteLine("{0} 请买车票!", i.CurrentItem());
                i.Next();
            }

            Console.Read();
        }
    }

{% endhighlight %}

### 示例
遍历公交车乘客买票

#### 使用内置的IEnumerator实现遍历

{% highlight java %}

     class Program
    {
        static void Main(string[] args)
        {

            IList<string> a = new List<string>();
            a.Add("大鸟");
            a.Add("小菜");
            a.Add("行李");
            a.Add("老外");
            a.Add("公交内部员工");
            a.Add("小偷");

            foreach (string item in a)
            {
                Console.WriteLine("{0} 请买车票!", item);
            }

            IEnumerator<string> e = a.GetEnumerator();
            while (e.MoveNext())
            {
                Console.WriteLine("{0} 请买车票!", e.Current);

            }
            Console.Read();
        }
    }

{% endhighlight %}
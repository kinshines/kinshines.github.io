---
layout: post
title: 泛型委托Action&lt;T&gt和Func&lt;TResult&gt;
author: kinshines
date:   2017-11-23
categories: c#
permalink: /archivers/generic-delegate-action-func
---

<p class="lead">本节笔者将介绍泛型委托Action&lt;T&gt;和Func&lt;TResult&gt;两类特殊的委托，这两个特殊的委托是Dot Net Framework自带的。结合lambda表达式，可以在写程序时，简洁代码和提高编码效率。</p>

### Action\<T\>和Func\<TResult\>两个委托的不同点
1. Action\<T\>只能委托必须是无返回值的方法
2. Func\<TResult\>只是委托必须有返回值的方法

### 代码示例
### Person类
{% highlight java %}
class Person
    {
        public Person(string name, int age)
        {
            this.instanceName = name;
            this.instanceAge = age;
        }

        private string instanceName;

        public string InstanceName
        {
            get { return instanceName; }
            set { instanceName = value; }
        }
        private int instanceAge;

        public int InstanceAge
        {
            get { return instanceAge; }
            set { instanceAge = value; }
        }

        public void DisplayName()
        {
            Console.WriteLine("Name:{0}", this.instanceName);
        }
        public void DisplayName(string name)
        {
            Console.WriteLine("Name:{0}", name);
        }
        public void DisplayAge(int age)
        {
            Console.WriteLine("Age:{0}", age);
        }
        public void DisplayNameAndAge(string name, int age)
        {
            Console.WriteLine(string.Format("Name:{0} And Age:{1} ", name, age));
        }
        public int GetAgeByName(string name)
        {
            if (name == instanceName)
            {
                return instanceAge;
            }
            else
            {
                return -1;
            }
        }
    }
{% endhighlight %}

### Program类

{% highlight java %}

class Program
    {
        //方法一:显式声明了一个委托,并将对 实例方法的引用分配给其委托实例。
        public delegate void ShowName();
        public delegate void ShowNameWithParameter(string name);
        public delegate void ShowAge(int age);
        public delegate void ShowNameAndAge(string name, int age);
        public delegate int ReturnName(string name);

        static void Main(string[] args)
        {
            #region Action<T>相关
            Person person = new Person("haima", 6);
            //非泛型委托
            //ShowName showName = new ShowName(name.DisplayName);
            //另一种写法
            ShowName showName = person.DisplayName;
            showName();
            ShowNameWithParameter showNameWithParameter = person.DisplayName;
            showNameWithParameter(person.InstanceName);
            ShowAge showAge = person.DisplayAge;
            showAge(person.InstanceAge);
            ShowNameAndAge showNameAndAge = person.DisplayNameAndAge;
            showNameAndAge(person.InstanceName, person.InstanceAge);

            //泛型委托Action<T,...>这里指多个类型参数
            //Action<T,..>委托：只能委托无返回值(void)的方法。
            //(2)可以使用此委托以参数形式传递方法，而不用显式声明自定义的委托 
            Action actionShowName = person.DisplayName;
            actionShowName();
            Action<string> actionShowName1 = person.DisplayName;
            actionShowName1(person.InstanceName);
            Action<int> actionShowAge = person.DisplayAge;
            actionShowAge(person.InstanceAge);
            Action<string, int> actionShowNameAndAge = person.DisplayNameAndAge;
            actionShowNameAndAge(person.InstanceName, person.InstanceAge);

            //匿名泛型委托
            //(1)初步
            Action<string> actionPrintNameAtFirst;
            actionPrintNameAtFirst = delegate (string s) { Console.WriteLine("Name:{0}", s); };
            actionPrintNameAtFirst(person.InstanceName);
            //(2)简化
            Action<string> actionprintName;
            actionprintName = s => Console.WriteLine("Name:{0}", s);
            actionprintName(person.InstanceName);

            //示例演示如何使用 Action<T> 委托来打印 List<T> 对象的内容
            //使用 Print 方法将列表的内容显示到控制台上。 此外，C# 示例还演示如何使用匿名方法将内容显示到控制台上
            //请注意该示例不显式声明 Action<T> 变量。 相反，它传递方法的引用，该方法采用单个参数而且不将值返回至 List<T>.ForEach 方法，其单个参数是一个 Action<T> 委托
            // 同样，在 C# 示例 中，Action<T> 委托不被显式地实例化，因为匿名方法的签名匹配 List<T>.ForEach 方法所期望的 Action<T> 委托的签名。
            List<String> names = new List<String>();
            names.Add("Bruce");
            names.Add("Alfred");
            names.Add("Tim");
            names.Add("Richard");
            names.ForEach(Print);
            names.ForEach(delegate (String name) { Console.WriteLine(name); });

            #endregion

            #region Fun<TResultf>相关
            //泛型委托Func<T,...,TResult> 委托,这里可以有一个或多个或者没有参数T,但必须有返回值并返回 TResult 参数所指定的类型的值
            //(1)必须有指定参数返回值。
            //(2)Lambda 表达式的基础类型是泛型 Func 委托之一。 这样能以参数形式传递 lambda 表达式，而不用显式将其分配给委托。
            //(3)因为 System.Linq 命名空间中许多类型方法具有 Func<T, TResult> 参数，因此可以给这些方法传递 lambda 表达式，而不用显式实例化 Func<T, TResult> 委托。

            //非泛型委托实现
            ReturnName returnName = person.GetAgeByName;
            Console.WriteLine("Age:{0}", returnName(person.InstanceName));

            //泛型委托实现
            Func<string, int> funReturnName = person.GetAgeByName;
            Console.WriteLine("Age:{0}", funReturnName(person.InstanceName));

            //泛型匿名委托实现
            //(1)初步
            Func<string, int> funReturnName1 = delegate (string s) { return person.GetAgeByName(s); };
            Console.WriteLine("Age:{0}", funReturnName1(person.InstanceName));
            //(2)优化
            Func<string, int> funReturnName2 = s => person.GetAgeByName(s);
            Console.WriteLine("Age:{0}", funReturnName2(person.InstanceName));

            //示例演示如何声明和使用 Func<T, TResult> 委托
            //此示例声明一个 Func<T, TResult> 变量，并为其分配了一个将字符串中的字符转换为大写的 lambda 表达式。
            //随后将封装此方法的委托传递给 Enumerable.Select 方法，以将字符串数组中的字符串更改为大写。 
            Func<string, string> selector = str => str.ToUpper();
            string[] words = { "orange", "apple", "Article", "elephant" };
            IEnumerable<String> aWords = words.Select(selector);
            foreach (String word in aWords)
                Console.WriteLine(word);

            #endregion
            Console.ReadKey();
        }

        private static void Print(string s)
        {
            Console.WriteLine(s);
        }
    }

{% endhighlight %}
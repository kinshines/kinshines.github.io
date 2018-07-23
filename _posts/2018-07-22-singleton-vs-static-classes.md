---
layout: post
title: 单例类(Singleton)对比静态类(Static)
author: kinshines
date:   2018-07-22
categories: c#
permalink: /archivers/singleton-vs-static-classes
---

<p class="lead">在.NET项目中，我们大都使用过静态类和单例类，有人会问，静态类能够提供单例类的功能，那使用单例类的意义是什么。或者至少会问，静态类和单例类的区别是什么，本文谈一下我的看法。
</p>

首先让我们看一下单例类，单例类保证你的程序里只有一个该类的实例，因此我们不能像普通类一样实例化它。先看一下下面一段代码

{% highlight java %}

using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
   
namespace SingletonVsStaticClass  
{  
    class SomeClass  
    {  
        public int ABC;  
    }  
   
    public sealed class SingletonSampleClass  
    {  
        static  SingletonSampleClass _instance;  
   
        public static SingletonSampleClass Instance  
        {  
            get { return _instance ?? (_instance = new SingletonSampleClass()); }  
        }  
        private SingletonSampleClass() { }  
   
        public string Message { get; set; }  
    }  
   
    static public class StaticSampleClass  
    {  
        private static readonly int SomeVariable;  
        //Static constructor is executed only once when the type is first used.   
        //All classes can have static constructors, not just static classes.  
        static StaticSampleClass()  
        {  
            SomeVariable = 1;  
            //Do the required things  
        }  
        public static string ShowValue()  
        {  
            return string.Format("The value of someVariable is {0}", SomeVariable);  
        }  
        public static string Message { get; set; }  
    }  
    class Program  
    {  
   
        static void Main(string[] args)  
        {  
            //Normal class instantiation and usage  
            var someClass = new SomeClass {ABC = 5};  
            Console.WriteLine("Normal class usage: "+someClass.ABC);  
   
            //Static Class instantiation  
            string returnValue = StaticSampleClass.ShowValue();  
            Console.WriteLine("Static class usage: " + returnValue);  
   
            //Singleton class instantiation  
            var singletonSampleClass = SingletonSampleClass.Instance;  
            singletonSampleClass.Message = "Hello";  
            Console.WriteLine("Singleton class usage: " + singletonSampleClass.Message);  
              
            //Test the instances if are equal  
            var anotherSingletonSampleClass = SingletonSampleClass.Instance;  
            Console.WriteLine("Checking if both instances are same: "+ singletonSampleClass.Equals(anotherSingletonSampleClass) );  
   
            Console.Read();  
        }  
    }  
}  

{% endhighlight %}

输出：

        Normal class usage:5
        Static class usage:The value of someVariable is 1
        Singleton class usage:Hello
        Checking if both instances are same:True

对于普通类，我们需要先创建实例，才能使用其方法和属性，对于静态类，则不需要创建实例即可使用其方法和属性，对于单例类，我们使用一个静态属性作为每一次的实例，无论我们创建多少次，它都返回相同的实例

### 何时使用单例类，何时使用静态类
静态类一般用于全局使用的单一实例和数据，它可以在任意时刻初始化，但通常使用懒加载，懒加载意味着尽可能晚的时刻初始化。静态类还有一个缺点，一旦它被装饰成static关键字，你就无法改变其行为了。

#### 单例类可以作为参数传递而静态类则不能
单例类不要求使用static关键字装饰，因此它可以像普通类一样作为方法的参数传递，但是静态类就不支持这样
{% highlight java %}

class SomeClass  
    {  
        public int ABC;  
   
        public void SingletonMethod(SingletonSampleClass singletonSampleClass)  
        {  
              
        }  
    }

    var someClass = new SomeClass {ABC = 5};  
    someClass.SingletonMethod(anotherSingletonSampleClass);  

{% endhighlight %}

#### 单例类支持接口继承而静态类则不能

{% highlight java %}

public interface IMyInterface  
   {  
       void MyInterfaceMethod();  
   }  
   class SomeClass : IMyInterface  
   {  
       public int ABC;  
  
       public void SingletonMethod(SingletonSampleClass singletonSampleClass)  
       {  
             
       }  
  
       public void MyInterfaceMethod()  
       {  
           throw new NotImplementedException();  
       }  
   }  
  
   public sealed class SingletonSampleClass : IMyInterface  
   {  
       static  SingletonSampleClass _instance;  
  
       public static SingletonSampleClass Instance  
       {  
           get { return _instance ?? (_instance = new SingletonSampleClass()); }  
       }  
       private SingletonSampleClass() { }  
  
       public string Message { get; set; }  
  
       public void MyInterfaceMethod()  
       {  
           throw new NotImplementedException();  
       }  
   }  
  
   static public class StaticSampleClass:IMyInterface  
   {  
       private static readonly int SomeVariable;  
       //Static constructor is executed only once when the type is first used.   
       //All classes can have static constructors, not just static classes.  
       static StaticSampleClass()  
       {  
           SomeVariable = 1;  
           //Do the required things  
       }  
       public static string ShowValue()  
       {  
           return string.Format("The value of someVariable is {0}", SomeVariable);  
       }  
       public static string Message { get; set; }  
   }  

{% endhighlight %}

以上代码会在编译时报错，原因是静态类不支持接口继承，
由于单例类支持接口继承，可以实现多态

使用单例类可以更灵活，因为单例类可以在依赖注入时选择注入类型为singleton 或 transient ，而静态类不支持依赖注入

想象一下以下场景，在你的应用程序里有一个全局共享的对象，你希望修改该类有多个实例，如果你使用的是单例类，你只需要修改构造函数为public即可，但是对于静态类的话，修改起来就不是那么容易了

静态类的优点在于，它在编译阶段就保证了不会有实例被意外添加进去，编译器会保证不会有实例的创建

单例类有一个私有的构造方法来防止类从外部被创建，单例类可以拥有一个实例成员，然而静态类也是做不到的

### 单例类线程安全初始化
为了保证单例类创建时的线程安全，在.NET中，通常会用以下方式设计单例：

{% highlight java %}

public sealed class Singleton  
{  
   private static readonly Singleton instance = new Singleton();  
    
   private Singleton(){}  
   
   public static Singleton Instance  
   {  
      get  
      {  
         return instance;  
      }  
   }  
}  

{% endhighlight %}

当然，还有更为通用的方式：

{% highlight java %}

using System;  
   
public sealed class Singleton  
{  
   private static volatile Singleton instance;  
   private static object syncRoot = new Object();  
   
   private Singleton() {}  
   
   public static Singleton Instance  
   {  
      get  
      {  
         if (instance == null)  
         {  
            lock (syncRoot)  
            {  
               if (instance == null)  
                  instance = new Singleton();  
            }  
         }  
   
         return instance;  
      }  
   }  
}

{% endhighlight %}

---
layout: post
title: 在LINQ to Entities 中构建谓词表达式 Predicates
author: kinshines
date:   2016-05-02
categories: c#
permalink: /archivers/Linq-to-Entities-Combining-Predicates
---

<p class="lead">本文将介绍在LINQ to Entities 中3种构建谓词表达式的方法</p>
测试类：
{% highlight java %}
public class Car
{
    public float Price { get; set; }
    public string Color { get; set; }
}
{% endhighlight %}

### 1.链式操作符 Chaining query operators
条件一：车是红色
条件二：车的价钱小于10.0
想要同时满足以上两个条件，可以使用Where方法链式操作：
{% highlight java %}
Expression<Func<Car, bool>> theCarIsRed = c => c.Color == “Red”;

Expression<Func<Car, bool>> theCarIsCheap = c => c.Price < 10.0;

IQueryable<Car> carQuery = …;

var query = carQuery.Where(theCarIsRed).Where(theCarIsCheap);
{% endhighlight %}

如果想要满足以上任意一个条件，可以使用Union方法：
{% highlight java %}
var query2 = carQuery.Where(theCarIsRed).Union(carQuery.Where(theCarIsCheap));
{% endhighlight %}
但是Union本身会带来效率问题，并且会消除结果中的一些重复项

### 2.使用原生API构建表达式 Build expressions manually
LINQ 表达式 API 包含一些工厂方法，我们可以手动调用：
{% highlight java %}
ParameterExpression c = Expression.Parameter(typeof(Car), “car”);

Expression theCarIsRed = Expression.Equal(Expression.Property(c, “Color”), Expression.Constant(“Red”));

Expression theCarIsCheap = Expression.LessThan(Expression.Property(c, “Price”), Expression.Constant(10.0));

Expression<Func<Car, bool>> theCarIsRedOrCheap = Expression.Lambda<Func<Car, bool>>(

    Expression.Or(theCarIsRed, theCarIsCheap), c);

var query = carQuery.Where(theCarIsRedOrCheap);
{% endhighlight %}

可以看到这种方式很不方便，写起来相当麻烦，因此我们将探索使用第3种方式

### 3.组合Lambda表达式
[C# 3.0 book](http://www.albahari.com/nutshell/)中介绍了一种组合Lambda表达式的方法：
{% highlight java %}
Expression<Func<Car, bool>> theCarIsRed = c1 => c1.Color == “Red”;

Expression<Func<Car, bool>> theCarIsCheap = c2 => c2.Price < 10.0;

Expression<Func<Car, bool>> theCarIsRedOrCheap = Expression.Lambda<Func<Car, bool>>(

    Expression.Or(theCarIsRed.Body, theCarIsCheap.Body), theCarIsRed.Parameters.Single());

var query = carQuery.Where(theCarIsRedOrCheap);
{% endhighlight %}

参照Matt Warren的系列文章[IQueryable Provider](https://blogs.msdn.microsoft.com/mattwar/2007/07/31/linq-building-an-iqueryable-provider-part-ii/) 中包含对[ExpressionVisitor](https://blogs.msdn.microsoft.com/mattwar/2007/07/30/linq-building-an-iqueryable-provider-part-i/) 的实现，我们可以重写表达式树，以下是实现参数重新绑定的类：

{% highlight java %}

public class ParameterRebinder : ExpressionVisitor {

    private readonly Dictionary<ParameterExpression, ParameterExpression> map;

 

    public ParameterRebinder(Dictionary<ParameterExpression, ParameterExpression> map) {

        this.map = map ?? new Dictionary<ParameterExpression, ParameterExpression>();

    }

 

    public static Expression ReplaceParameters(Dictionary<ParameterExpression, ParameterExpression> map, Expression exp) {

        return new ParameterRebinder(map).Visit(exp);

    }

 

    protected override Expression VisitParameter(ParameterExpression p) {

        ParameterExpression replacement;

        if (map.TryGetValue(p, out replacement)) {

            p = replacement;

        }

        return base.VisitParameter(p);

    }

}
{% endhighlight %}

现在，我们可以写一个工具类在不使用invoke的情况下实现组合lambda表达式，并且提供EF友好的And和Or方法：
{% highlight java %}

public static class Utility {

    public static Expression<T> Compose<T>(this Expression<T> first, Expression<T> second, Func<Expression, Expression, Expression> merge) {

        // build parameter map (from parameters of second to parameters of first)

        var map = first.Parameters.Select((f, i) => new { f, s = second.Parameters[i] }).ToDictionary(p => p.s, p => p.f);

 

        // replace parameters in the second lambda expression with parameters from the first

        var secondBody = ParameterRebinder.ReplaceParameters(map, second.Body);

 

        // apply composition of lambda expression bodies to parameters from the first expression 

        return Expression.Lambda<T>(merge(first.Body, secondBody), first.Parameters);

    }

 

    public static Expression<Func<T, bool>> And<T>(this Expression<Func<T, bool>> first, Expression<Func<T, bool>> second) {

        return first.Compose(second, Expression.And);

    }

 

    public static Expression<Func<T, bool>> Or<T>(this Expression<Func<T, bool>> first, Expression<Func<T, bool>> second) {

        return first.Compose(second, Expression.Or);

    }

}

{% endhighlight %}


使用lambda表达式组合，我们可以这样写：

{% highlight java %}
Expression<Func<Car, bool>> theCarIsRed = c => c.Color == “Red”;

Expression<Func<Car, bool>> theCarIsCheap = c => c.Price < 10.0;

Expression<Func<Car, bool>> theCarIsRedOrCheap = theCarIsRed.Or(theCarIsCheap);

var query = carQuery.Where(theCarIsRedOrCheap);
{% endhighlight %}
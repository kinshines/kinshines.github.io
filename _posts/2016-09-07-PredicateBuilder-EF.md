---
layout: post
title: PredicateBuilder 对 Entity Framework 支持的改进
author: kinshines
date:   2016-09-07
categories: ef
permalink: /archivers/PredicateBuilder-EF
---

<p class="lead"> 曾几何时，网络上曾经大传 <a href="https://bytes.com/topic/net/insights/887941-dynamic-linq-where-clauses-predicatebuilder">PredicateBuilder</a> 用于拼接两个 Lambda 表达式树。在对内存数据的筛选上面，其简洁方便的操作大放异彩，但是对数据库操作的不支持，一直是其硬伤。PredicateBuilder 拼接表达式的过程中，产生的 Invoke 表达式无法翻译成 SQL 语句，这是其根本原因。另外，Invoke 表达式编译后，形成的委托调用委托的方式，也是对性能的一种损耗。</p>

当然，也有很多人对其做过改造，不过给人的感觉，总不是那么完美。以前，因为种种原因，我们不得不忍受这种问题。前几天，由于工作中用到了 Entity Framework 和动态拼接 Lambda 表达式树，为了避免全表查询的尴尬，硬着头皮研究改造 PredicateBuilder —— 各种纠结。还好，了解透彻原理后就很简单了。期间，用到了下面这个测试类：

{% highlight java %}
public class Person
{
    public int ID { get; set; }
    public string Name { get; set; }
}
{% endhighlight %}

这里先说一下要达到的目的：

两个表达式：

{% highlight java %}
Expression<Func<Person, bool>> a = p => p.ID == 3;
Expression<Func<Person, bool>> b = c => c.Name == "张三";
{% endhighlight %}
拼接之后，想要达成如下形式：

{% highlight java %}
Expression<Func<Person, bool>> x = y => y.ID == 3 && y.Name == "张三";
{% endhighlight %}

原始 PredicateBuilder 用的是 Invoke 表达式，也就是说没有解析待拼接的表达式。我们可以解析一下，看看能否达到效果。

首先来看相同点：

表达式 a ：根是一个相等判断的二元表达式，其左子树是属性的调用的表达式，右子树是一个数字常量表达式。

表达式 b ：根是一个相等判断的二元表达式，其左子树是属性的调用的表达式，右子树是一个数字常量表达式。

下面看不同：

表达式 a 参数名字是 p ，表达式 b 参数名字是 c —— 两者参数的名称不同，但是类型是相同的。

对于目标表达式，这里我第一印象是把右边表达式的参数，改成左边的，然后把左右两边表达式的 Body 拼接，加上左边的参数，这就是目标表达式树。经过分析，这种做法是可行的，下面上代码，代码不解释：
{% highlight java %}
    /// <summary>
    /// 谓词表达式构建器
    /// </summary>
    public static class PredicateBuilder
    {
        #region Expression Joiner
        /// <summary>
        /// 创建一个值恒为 <c>true</c> 的表达式。
        /// </summary>
        /// <typeparam name="T">表达式方法类型</typeparam>
        /// <returns>一个值恒为 <c>true</c> 的表达式。</returns>
        public static Expression<Func<T, bool>> True<T>() { return p => true; }

        /// <summary>
        /// 创建一个值恒为 <c>false</c> 的表达式。
        /// </summary>
        /// <typeparam name="T">表达式方法类型</typeparam>
        /// <returns>一个值恒为 <c>false</c> 的表达式。</returns>
        public static Expression<Func<T, bool>> False<T>() { return f => false; }

        /// <summary>
        /// 使用 Expression.OrElse 的方式拼接两个 System.Linq.Expression。
        /// </summary>
        /// <typeparam name="T">表达式方法类型</typeparam>
        /// <param name="left">左边的 System.Linq.Expression 。</param>
        /// <param name="right">右边的 System.Linq.Expression。</param>
        /// <returns>拼接完成的 System.Linq.Expression。</returns>
        public static Expression<T> Or<T>(this Expression<T> left, Expression<T> right)
        {
            return MakeBinary(left, right, Expression.OrElse);
        }

        /// <summary>
        /// 使用 Expression.AndAlso 的方式拼接两个 System.Linq.Expression。
        /// </summary>
        /// <typeparam name="T">表达式方法类型</typeparam>
        /// <param name="left">左边的 System.Linq.Expression 。</param>
        /// <param name="right">右边的 System.Linq.Expression。</param>
        /// <returns>拼接完成的 System.Linq.Expression。</returns>
        public static Expression<T> And<T>(this Expression<T> left, Expression<T> right)
        {
            return MakeBinary(left, right, Expression.AndAlso);
        }

        /// <summary>
        /// 使用自定义的方式拼接两个 System.Linq.Expression。
        /// </summary>
        /// <typeparam name="T">表达式方法类型</typeparam>
        /// <param name="left">左边的 System.Linq.Expression 。</param>
        /// <param name="right">右边的 System.Linq.Expression。</param>
        /// <returns>拼接完成的 System.Linq.Expression。</returns>
        public static Expression<T> MakeBinary<T>(this Expression<T> left, Expression<T> right, Func<Expression, Expression, Expression> func)
        {
            return MakeBinary((LambdaExpression)left, right, func) as Expression<T>;
        }

        /// <summary>
        /// 拼接两个 <paramref name="System.Linq.Expression"/> ，两个 <paramref name="System.Linq.Expression"/> 的参数必须完全相同。
        /// </summary>
        /// <typeparam name="T">表达式中的元素类型</typeparam>
        /// <param name="left">左边的 <paramref name="System.Linq.Expression"/></param>
        /// <param name="right">右边的 <paramref name="System.Linq.Expression"/></param>
        /// <param name="func">表达式拼接的具体逻辑</param>
        /// <returns>拼接完成的 <paramref name="System.Linq.Expression"/></returns>
        public static LambdaExpression MakeBinary(this LambdaExpression left, LambdaExpression right, Func<Expression, Expression, Expression> func)
        {
            var data = Combinate(right.Parameters, left.Parameters).ToArray();
            right = ParameterReplace.Replace(right, data) as LambdaExpression;
            return Expression.Lambda(func(left.Body, right.Body), left.Parameters.ToArray());
        }
        #endregion

        #region Private Methods
        private static IEnumerable<KeyValuePair<T, T>> Combinate<T>(IEnumerable<T> left, IEnumerable<T> right)
        {
            var a = left.GetEnumerator();
            var b = right.GetEnumerator();
            while (a.MoveNext() && b.MoveNext())
                yield return new KeyValuePair<T, T>(a.Current, b.Current);
        }
        #endregion
    }

    #region class: ParameterReplace
    internal sealed class ParameterReplace : ExpressionVisitor
    {
        public static Expression Replace(Expression e, IEnumerable<KeyValuePair<ParameterExpression, ParameterExpression>> paramList)
        {
            var item = new ParameterReplace(paramList);
            return item.Visit(e);
        }

        private Dictionary<ParameterExpression, ParameterExpression> parameters = null;

        public ParameterReplace(IEnumerable<KeyValuePair<ParameterExpression, ParameterExpression>> paramList)
        {
            parameters = paramList.ToDictionary(p => p.Key, p => p.Value, new ParameterEquality());
        }

        protected override Expression VisitParameter(ParameterExpression p)
        {
            ParameterExpression result;
            if (parameters.TryGetValue(p, out result))
                return result;
            else
                return base.VisitParameter(p);
        }

        #region class: ParameterEquality
        private class ParameterEquality : IEqualityComparer<ParameterExpression>
        {
            public bool Equals(ParameterExpression x, ParameterExpression y)
            {
                if (x == null || y == null)
                    return false;

                return x.Type == y.Type;
            }

            public int GetHashCode(ParameterExpression obj)
            {
                if (obj == null)
                    return 0;

                return obj.Type.GetHashCode();
            }
        }
        #endregion
    }
    #endregion
{% endhighlight %}


如果是在 .Net 3.5 下运行的话，还少一个 ExpressionVisitor 类，在 .Net 4.0 就不需要了，顺便把代码贴上来，需要的就拿走吧：
{% highlight java %}
#region class: ExpressionVisitor
[DebuggerStepThrough]
public abstract class ExpressionVisitor
{
    protected ExpressionVisitor()
    {
    }

    protected virtual Expression Visit(Expression exp)
    {
        if (exp == null)
            return exp;
        switch (exp.NodeType)
        {
            case ExpressionType.Negate:
            case ExpressionType.NegateChecked:
            case ExpressionType.Not:
            case ExpressionType.Convert:
            case ExpressionType.ConvertChecked:
            case ExpressionType.ArrayLength:
            case ExpressionType.Quote:
            case ExpressionType.TypeAs:
                return this.VisitUnary((UnaryExpression)exp);
            case ExpressionType.Add:
            case ExpressionType.AddChecked:
            case ExpressionType.Subtract:
            case ExpressionType.SubtractChecked:
            case ExpressionType.Multiply:
            case ExpressionType.MultiplyChecked:
            case ExpressionType.Divide:
            case ExpressionType.Modulo:
            case ExpressionType.And:
            case ExpressionType.AndAlso:
            case ExpressionType.Or:
            case ExpressionType.OrElse:
            case ExpressionType.LessThan:
            case ExpressionType.LessThanOrEqual:
            case ExpressionType.GreaterThan:
            case ExpressionType.GreaterThanOrEqual:
            case ExpressionType.Equal:
            case ExpressionType.NotEqual:
            case ExpressionType.Coalesce:
            case ExpressionType.ArrayIndex:
            case ExpressionType.RightShift:
            case ExpressionType.LeftShift:
            case ExpressionType.ExclusiveOr:
                return this.VisitBinary((BinaryExpression)exp);
            case ExpressionType.TypeIs:
                return this.VisitTypeIs((TypeBinaryExpression)exp);
            case ExpressionType.Conditional:
                return this.VisitConditional((ConditionalExpression)exp);
            case ExpressionType.Constant:
                return this.VisitConstant((ConstantExpression)exp);
            case ExpressionType.Parameter:
                return this.VisitParameter((ParameterExpression)exp);
            case ExpressionType.MemberAccess:
                return this.VisitMemberAccess((MemberExpression)exp);
            case ExpressionType.Call:
                return this.VisitMethodCall((MethodCallExpression)exp);
            case ExpressionType.Lambda:
                return this.VisitLambda((LambdaExpression)exp);
            case ExpressionType.New:
                return this.VisitNew((NewExpression)exp);
            case ExpressionType.NewArrayInit:
            case ExpressionType.NewArrayBounds:
                return this.VisitNewArray((NewArrayExpression)exp);
            case ExpressionType.Invoke:
                return this.VisitInvocation((InvocationExpression)exp);
            case ExpressionType.MemberInit:
                return this.VisitMemberInit((MemberInitExpression)exp);
            case ExpressionType.ListInit:
                return this.VisitListInit((ListInitExpression)exp);
            default:
                throw new Exception(string.Format("Unhandled expression type: '{0}'", exp.NodeType));
        }
    }

    protected virtual MemberBinding VisitBinding(MemberBinding binding)
    {
        switch (binding.BindingType)
        {
            case MemberBindingType.Assignment:
                return this.VisitMemberAssignment((MemberAssignment)binding);
            case MemberBindingType.MemberBinding:
                return this.VisitMemberMemberBinding((MemberMemberBinding)binding);
            case MemberBindingType.ListBinding:
                return this.VisitMemberListBinding((MemberListBinding)binding);
            default:
                throw new Exception(string.Format("Unhandled binding type '{0}'", binding.BindingType));
        }
    }

    protected virtual ElementInit VisitElementInitializer(ElementInit initializer)
    {
        ReadOnlyCollection<Expression> arguments = this.VisitExpressionList(initializer.Arguments);
        if (arguments != initializer.Arguments)
        {
            return Expression.ElementInit(initializer.AddMethod, arguments);
        }
        return initializer;
    }

    protected virtual Expression VisitUnary(UnaryExpression u)
    {
        Expression operand = this.Visit(u.Operand);
        if (operand != u.Operand)
        {
            return Expression.MakeUnary(u.NodeType, operand, u.Type, u.Method);
        }
        return u;
    }

    protected virtual Expression VisitBinary(BinaryExpression b)
    {
        Expression left = this.Visit(b.Left);
        Expression right = this.Visit(b.Right);
        Expression conversion = this.Visit(b.Conversion);
        if (left != b.Left || right != b.Right || conversion != b.Conversion)
        {
            if (b.NodeType == ExpressionType.Coalesce && b.Conversion != null)
                return Expression.Coalesce(left, right, conversion as LambdaExpression);
            else
                return Expression.MakeBinary(b.NodeType, left, right, b.IsLiftedToNull, b.Method);
        }
        return b;
    }

    protected virtual Expression VisitTypeIs(TypeBinaryExpression b)
    {
        Expression expr = this.Visit(b.Expression);
        if (expr != b.Expression)
        {
            return Expression.TypeIs(expr, b.TypeOperand);
        }
        return b;
    }

    protected virtual Expression VisitConstant(ConstantExpression c)
    {
        return c;
    }

    protected virtual Expression VisitConditional(ConditionalExpression c)
    {
        Expression test = this.Visit(c.Test);
        Expression ifTrue = this.Visit(c.IfTrue);
        Expression ifFalse = this.Visit(c.IfFalse);
        if (test != c.Test || ifTrue != c.IfTrue || ifFalse != c.IfFalse)
        {
            return Expression.Condition(test, ifTrue, ifFalse);
        }
        return c;
    }

    protected virtual Expression VisitParameter(ParameterExpression p)
    {
        return p;
    }

    protected virtual Expression VisitMemberAccess(MemberExpression m)
    {
        Expression exp = this.Visit(m.Expression);
        if (exp != m.Expression)
        {
            return Expression.MakeMemberAccess(exp, m.Member);
        }
        return m;
    }

    protected virtual Expression VisitMethodCall(MethodCallExpression m)
    {
        Expression obj = this.Visit(m.Object);
        IEnumerable<Expression> args = this.VisitExpressionList(m.Arguments);
        if (obj != m.Object || args != m.Arguments)
        {
            return Expression.Call(obj, m.Method, args);
        }
        return m;
    }

    protected virtual ReadOnlyCollection<Expression> VisitExpressionList(ReadOnlyCollection<Expression> original)
    {
        List<Expression> list = null;
        for (int i = 0, n = original.Count; i < n; i++)
        {
            Expression p = this.Visit(original[i]);
            if (list != null)
            {
                list.Add(p);
            }
            else if (p != original[i])
            {
                list = new List<Expression>(n);
                for (int j = 0; j < i; j++)
                {
                    list.Add(original[j]);
                }
                list.Add(p);
            }
        }
        if (list != null)
        {
            return list.AsReadOnly();
        }
        return original;
    }

    protected virtual MemberAssignment VisitMemberAssignment(MemberAssignment assignment)
    {
        Expression e = this.Visit(assignment.Expression);
        if (e != assignment.Expression)
        {
            return Expression.Bind(assignment.Member, e);
        }
        return assignment;
    }

    protected virtual MemberMemberBinding VisitMemberMemberBinding(MemberMemberBinding binding)
    {
        IEnumerable<MemberBinding> bindings = this.VisitBindingList(binding.Bindings);
        if (bindings != binding.Bindings)
        {
            return Expression.MemberBind(binding.Member, bindings);
        }
        return binding;
    }

    protected virtual MemberListBinding VisitMemberListBinding(MemberListBinding binding)
    {
        IEnumerable<ElementInit> initializers = this.VisitElementInitializerList(binding.Initializers);
        if (initializers != binding.Initializers)
        {
            return Expression.ListBind(binding.Member, initializers);
        }
        return binding;
    }

    protected virtual IEnumerable<MemberBinding> VisitBindingList(ReadOnlyCollection<MemberBinding> original)
    {
        List<MemberBinding> list = null;
        for (int i = 0, n = original.Count; i < n; i++)
        {
            MemberBinding b = this.VisitBinding(original[i]);
            if (list != null)
            {
                list.Add(b);
            }
            else if (b != original[i])
            {
                list = new List<MemberBinding>(n);
                for (int j = 0; j < i; j++)
                {
                    list.Add(original[j]);
                }
                list.Add(b);
            }
        }
        if (list != null)
            return list;
        return original;
    }

    protected virtual IEnumerable<ElementInit> VisitElementInitializerList(ReadOnlyCollection<ElementInit> original)
    {
        List<ElementInit> list = null;
        for (int i = 0, n = original.Count; i < n; i++)
        {
            ElementInit init = this.VisitElementInitializer(original[i]);
            if (list != null)
            {
                list.Add(init);
            }
            else if (init != original[i])
            {
                list = new List<ElementInit>(n);
                for (int j = 0; j < i; j++)
                {
                    list.Add(original[j]);
                }
                list.Add(init);
            }
        }
        if (list != null)
            return list;
        return original;
    }

    protected virtual Expression VisitLambda(LambdaExpression lambda)
    {
        Expression body = this.Visit(lambda.Body);
        if (body != lambda.Body)
        {
            return Expression.Lambda(lambda.Type, body, lambda.Parameters);
        }
        return lambda;
    }

    protected virtual NewExpression VisitNew(NewExpression nex)
    {
        IEnumerable<Expression> args = this.VisitExpressionList(nex.Arguments);
        if (args != nex.Arguments)
        {
            if (nex.Members != null)
                return Expression.New(nex.Constructor, args, nex.Members);
            else
                return Expression.New(nex.Constructor, args);
        }
        return nex;
    }

    protected virtual Expression VisitMemberInit(MemberInitExpression init)
    {
        NewExpression n = this.VisitNew(init.NewExpression);
        IEnumerable<MemberBinding> bindings = this.VisitBindingList(init.Bindings);
        if (n != init.NewExpression || bindings != init.Bindings)
        {
            return Expression.MemberInit(n, bindings);
        }
        return init;
    }

    protected virtual Expression VisitListInit(ListInitExpression init)
    {
        NewExpression n = this.VisitNew(init.NewExpression);
        IEnumerable<ElementInit> initializers = this.VisitElementInitializerList(init.Initializers);
        if (n != init.NewExpression || initializers != init.Initializers)
        {
            return Expression.ListInit(n, initializers);
        }
        return init;
    }

    protected virtual Expression VisitNewArray(NewArrayExpression na)
    {
        IEnumerable<Expression> exprs = this.VisitExpressionList(na.Expressions);
        if (exprs != na.Expressions)
        {
            if (na.NodeType == ExpressionType.NewArrayInit)
            {
                return Expression.NewArrayInit(na.Type.GetElementType(), exprs);
            }
            else
            {
                return Expression.NewArrayBounds(na.Type.GetElementType(), exprs);
            }
        }
        return na;
    }

    protected virtual Expression VisitInvocation(InvocationExpression iv)
    {
        IEnumerable<Expression> args = this.VisitExpressionList(iv.Arguments);
        Expression expr = this.Visit(iv.Expression);
        if (args != iv.Arguments || expr != iv.Expression)
        {
            return Expression.Invoke(expr, args);
        }
        return iv;
    }
}
#endregion
{% endhighlight %}
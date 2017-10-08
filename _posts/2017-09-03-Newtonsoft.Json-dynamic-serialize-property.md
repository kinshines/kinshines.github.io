---
layout: post
title: Newtonsoft Json.NET动态决定属性是否序列化
author: kinshines
date:   2017-09-03
categories: c#
permalink: /archivers/Newtonsoft-Json-dynamic-serialize-property
---

<p class="lead">今天遇到一个需求，将一个类序列化时，需要动态决定哪些属性是否执行序列化。Json.NET作为一款强大的json序列化工具，提供ContractResolver以实现神乎奇技的高度动态化</p>

下面这个范例，展示两种动态决定应序列化属性的情境:

> JSerialize时传入属性名称数组作为参数，指定JSON应包含的属性。由对象属性值决定属性是否要序列化，例如: 如果是女生就不包含年龄。(这几乎已弹性到极点，虽然实务上不常用到)

程序的做法是提供两个继承自DefaultContractResolver的类: 

* LimitPropsContractResolver在建构时传入string[]参数列出要序列化的属性名称，并覆写CreateProperties方法，过滤base.CreateProperties()传回的IList<JsonProperty>，只保留前述string[]有列出的属性；

* HideAgeContractResolver则覆写CreateProperty()方法，由base.CreateProperty()取得JsonProperty，JsonProperty有个ShouldSerialize属性可以传入Lambda表达式，遍历处理每个要序列化的对象，在Lambda表达式中可将对象转型为原型别进行判断，若不要序列化就返回false

下面是代码演示：

{% highlight java %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Text;
using Newtonsoft.Json;
using Newtonsoft.Json.Converters;
using Newtonsoft.Json.Serialization;
 
namespace ConsoleApplication1
{
    class Program
    {
        public enum Gender 
        {
            Male, Female
        }
 
        public class Person
        {
            public string Name { get; set; }
            [JsonConverter(typeof(StringEnumConverter))]
            public Gender Gender { get; set; }
            public int Age { get; set; }
            public Person(string name, Gender gender, int age)
            {
                Name = name; Gender = gender; Age = age;
            }
        }
 
        public class HideAgeContractResolver : DefaultContractResolver
        {
            //REF: http://james.newtonking.com/projects/json/help/index.html?topic=html/ContractResolver.htm
            protected override JsonProperty CreateProperty(MemberInfo member, 
                MemberSerialization memberSerialization)
            {
                JsonProperty p = base.CreateProperty(member, memberSerialization);
                if (p.PropertyName == "Age")
                {
                    //依性别决定是否要序列化
                    p.ShouldSerialize = instance =>
                    {
                        Person person = (Person)instance;
                        return person.Gender == Gender.Male;
                    };
                }
                return p;
            }
        }
 
        public class LimitPropsContractResolver : DefaultContractResolver
        {
            string[] props = null;
            bool retain;

            public LimitPropsContractResolver(string[] props,bool retain=true)
            {
                //指定要序列化属性的清单
                this.props = props;
                this.retain = retain;
            }
            //REF: http://james.newtonking.com/archive/2009/10/23/efficient-json-with-json-net-reducing-serialized-json-size.aspx
            protected override IList<JsonProperty> CreateProperties(Type type, MemberSerialization memberSerialization)
        {
            IList<JsonProperty> list = base.CreateProperties(type, memberSerialization);
            //只保留清单有列出的属性
            return list.Where(p =>
            {
                if (retain)
                {
                    return props.Contains(p.PropertyName);
                }
                else
                {
                    return !props.Contains(p.PropertyName);
                }
            }).ToList();
        }
 
        }
 
        static void Main(string[] args)
        {
            List<Person> list = new List<Person>();
            list.Add(new Person("George", Gender.Male, 18));
            list.Add(new Person("Mary", Gender.Female, 40));
            //正常输出
            Console.WriteLine(JsonConvert.SerializeObject(
                list, Formatting.Indented));
            var settings = new JsonSerializerSettings();
            //加上ContractResolver，正向表列哪些属性要序列化
            settings.ContractResolver = 
                new LimitPropsContractResolver("Name,Age".Split(','));
            Console.WriteLine(JsonConvert.SerializeObject(
                list, Formatting.Indented, settings));
            //加上ContractResolver，依对象的属性值动态决定要不要序列化
            settings.ContractResolver = new HideAgeContractResolver();
            Console.WriteLine(JsonConvert.SerializeObject(
                list, Formatting.Indented, settings));
            Console.ReadLine();
 
        }
    }
}

{% endhighlight %}


程序执行结果如下，共有三段输出，第一段为正常版；第二段套用LimitPropsContractResolver("Name,Age".Split(','))，故JSON中只见Name及Age，Gender被隐藏；第三段套用了HideAgeContractResolver()，如结果所示，Mary的JSON内容不包含年龄，George则包含。

{% highlight json %}
[ 
{ 
"Name": "George", 
"Gender": "Male", 
"Age": 18 
}, 
{ 
"Name": "Mary", 
"Gender": "Female", 
"Age": 40 
} 
] 
[ 
{ 
"Name": "George", 
"Age": 18 
}, 
{ 
"Name": "Mary", 
"Age": 40 
} 
] 
[ 
{ 
"Name": "George", 
"Gender": "Male", 
"Age": 18 
}, 
{ 
"Name": "Mary", 
"Gender": "Female" 
} 
] 

{% endhighlight %}
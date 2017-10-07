---
layout: post
title: C# 中 Datatable.Select()用法简介
author: kinshines
date:   2016-09-13
categories: c#
permalink: /archivers/csharp-datatable-select
---

<p class="lead">DataTable是我们在进行开发时经常用到的一个类，并且经常需要对DataTable中的数据进行筛选等操作，下面就介绍一下Datatable中经常用到的一个方法——Select</p>

微软提供了四个函数的重载，分别是

                Select()

                Select(string filterExpression)

                Select(string filterExpression, string sort)

                Select(string filterExpression,string sort, DataViewRowState record States)

1.  Select()——获取所有 System.Data.DataRow 对象的数组。

2.  Select(string filterExpression)——按照主键顺序（如果没有主键，则按照添加顺序）获取与筛选条件相匹配的所有 System.Data.DataRow 对象的数组。

3.  Select(string filterExpression, string sort)——获取按照指定的排序顺序且与筛选条件相匹配的所有 System.Data.DataRow 对象的数组。

4.  Select(string filterExpression, string sort, DataViewRowState recordStates)——获取与排序顺序中的筛选器以及指定的状态相匹配的所有 System.Data.DataRow 对象的数组。

下面是对这些方法进行演示的示例：

{% highlight java %}
using System;

using System.Collections.Generic;

using System.Text;

using System.Data;

 

namespace TestDataTableSelect

{

    class Program

    {

        static DataTable dt = new DataTable();

        static void Main(string[] args)

        {         

            DataColumn dc1 = new DataColumn("id");

            dc1.DataType=typeof(int);

            DataColumn dc2 = new DataColumn("name");

            dc2.DataType=typeof(System.String);

            dt.Columns.Add(dc1);

            dt.Columns.Add(dc2);

            for (int i = 1; i <=10;i++ )

            {

                DataRow dr = dt.NewRow();

                if (i <= 5)

                {

                    dr[0] = i;

                    dr[1] = i + "--" + "hello";

                }

                else

                {

                    dr[0] = i;

                    dr[1] = i + "--" + "nihao";

                }

                dt.Rows.Add(dr);

            }

 

            Select();

            Select("id>='3' and name='3--hello'");//支持and

            Select("id>='3' or id='1'");//支持or

            Select("name like '%hello%'");//支持like   

            Select("id>5","id desc");

            Select("id>5", "id desc",DataViewRowState.Added);

        }

 

        private static void Select()

        {

            DataRow[] arrayDR = dt.Select();

            foreach(DataRow dr in arrayDR)

            {

                Console.WriteLine(dr[0].ToString()+"    "+dr[1].ToString());

            }

            Console.ReadLine();

        }

 

        private static void Select(string filterExpression)

        {

            DataRow[] arrayDR = dt.Select(filterExpression);

            foreach (DataRow dr in arrayDR)

            {

                Console.WriteLine(dr[0].ToString() + "    " + dr[1].ToString());

            }

            Console.ReadLine();

        }

 

        private static void Select(string filterExpression, string sort)

        {

            DataRow[] arrayDR = dt.Select(filterExpression,sort);

            foreach (DataRow dr in arrayDR)

            {

                Console.WriteLine(dr[0].ToString() + "    " + dr[1].ToString());

            }

            Console.ReadLine();

        }

 

        private static void Select(string filterExpression, string sort, DataViewRowState recordStates)

        {

            DataRow[] arrayDR = dt.Select(filterExpression, sort,recordStates);

            foreach (DataRow dr in arrayDR)

            {

                Console.WriteLine(dr[0].ToString() + "    " + dr[1].ToString());

            }

            Console.ReadLine();

        }

    }

}
{% endhighlight %}

注意事项：上面的Select操作是大小写不敏感的（记录的字段不敏感），如果需要区分大小写，需要将DataTable的caseSensitive属性设为true。
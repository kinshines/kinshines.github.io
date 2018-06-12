---
layout: post
title: Entity Framework 并发处理
author: kinshines
date:   2018-06-12
categories: ef
permalink: /archivers/ef-concurrency-pattern
---

<p class="lead">本篇将介绍关于Entity Framework 的并发处理
</p>

### 并发
什么是并发？


并发分悲观并发和乐观并发。

#### 悲观并发
比如有两个用户A,B，同时登录系统修改一个文档，如果A先进入修改，则系统会把该文档锁住，B就没办法打开了，只有等A修改完，完全退出的时候B才能进入修改。
#### 乐观并发
同上面的例子，A,B两个用户同时登录，如果A先进入修改紧跟着B也进入了。A修改文档的同时B也在修改。如果在A保存之后B再保存他的修改，此时系统检测到数据库中文档记录与B刚进入时不一致，B保存时会抛出异常，修改失败。

EF中如何控制并发？

Entity Framework不支持悲观并发，只支持乐观并发。

#### 对整张表做并发处理
如果要对某一个表做并发处理，就在该表中加一条Timestamp类型的字段。注意，一张表中只能有一个Timestamp的字段。

Data Annotations中用Timestamp来标识设置并发控制字段，标识为Timestamp的字段必需为byte[]类型。

{% highlight java %}

public class Person
    {
        public int PersonId { get; set; }
        public int SocialSecurityNumber { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        [Timestamp]
        public byte[] RowVersion { get; set; }
    }

{% endhighlight %}

Fluent API用IsRowVersion方法

        modelBuilder.Entity<Person>().Property(p => p.RowVersion).IsRowVersion();

SQL Server生成的数据库中，RowVersion是timestamp类型，而MySQL数据库中，RowVersion是longblob类型

下面我们写一段代码来测试一下：

{% highlight java %}

static void Main(string[] args)
        {
            var person = new Person
            {
                FirstName = "Rowan",
                LastName = "Miller",
                SocialSecurityNumber = 12345678
            };
            //新增一条记录，保存到数据库中
            using (var con = new BreakAwayContext())
            {
                con.People.Add(person);
                con.SaveChanges();
            }

            var firContext = new BreakAwayContext();
            //取第一条记录,并修改一个字段：这里是修改了FirstName
            //先不保存
            var p1 = firContext.People.FirstOrDefault();
            p1.FirstName = "Steven";

            //再创建一个Context，同样取第一条记录，修改LastName字段并保存
            using (var secContext = new BreakAwayContext())
            {
                var p2 = secContext.People.FirstOrDefault();
                p2.LastName = "Francis";
                secContext.SaveChanges();
            }
            try
            {
                firContext.SaveChanges();
                Console.WriteLine(" 保存成功");
            }
            catch (DbUpdateConcurrencyException ex)
            {
                Console.WriteLine(ex.Entries.First().Entity.GetType().Name + " 保存失败");
            }
            Console.Read();
        }

{% endhighlight %}

上面我们实例化了三个DbContext,第一个增加一条记录到数据库中，第二个修改刚增加的记录但不保存，然后第三个Context也取刚新增的记录并保存，最后再保存第二个Context，结果保存失败。

分析EF生成的SQL语句：

        exec sp_executesql N'update [dbo].[People] 
        set [LastName] = @0 where (([PersonId] = @1) and ([RowVersion] = @2))select [RowVersion] from [dbo].[People] where @@ROWCOUNT > 0 and [PersonId] = @1',N'@0 nvarchar(max) ,@1 int,@2 binary(8)',@0=N'Francis',@1=1,@2=0x00000000000007D1

可以看到，它在取对应记录的时候把RowVersion也作为筛选条件。上面例子中的secContext保存的时候，数据库中的RowVersion字段的值就变了，所以firContext保存的时候用原来的RowVersion取值，自然就取不到相应的记录而报错。

#### 对某个字段做并发处理
Data Annotations中用ConcurrencyCheck来标识
{% highlight java %}

public class Person
    {
        public int PersonId { get; set; }
        public int SocialSecurityNumber { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        [Timestamp]
        public byte[] RowVersion { get; set; }
    }

{% endhighlight %}

Fluent API用IsConcurrencyToken方法

        modelBuilder.Entity<Person>().Property(p => p.SocialSecurityNumber).IsConcurrencyToken();

上面的实体中，我们将SocialSecurityNumber(社会保险号)标识为开放式并发，也写一个类似的代码测试一下：

{% highlight java %}

static void Main(string[] args)
        {
            var person = new Person
            {
                FirstName = "Rowan",
                LastName = "Miller",
                SocialSecurityNumber = 12345678
            };
            //新增一条记录，保存到数据库中
            using (var con = new BreakAwayContext())
            {
                con.People.Add(person);
                con.SaveChanges();
            }

            var firContext = new BreakAwayContext();
            //取第一条记录,并修改SocialSecurityNumber字段
            //先不保存
            var p1 = firContext.People.FirstOrDefault();
            p1.SocialSecurityNumber = 123;

            //再创建一个Context，同样取第一条记录，
            //修改SocialSecurityNumber字段并保存
            using (var secContext = new BreakAwayContext())
            {
                var p2 = secContext.People.FirstOrDefault();
                p2.SocialSecurityNumber = 456;
                secContext.SaveChanges();
            }
            try
            {
                firContext.SaveChanges();
                Console.WriteLine(" 保存成功");
            }
            catch (DbUpdateConcurrencyException ex)
            {
                Console.WriteLine(ex.Entries.First().Entity.GetType().Name + " 保存失败");
            }
            Console.Read();
        }

{% endhighlight %}

运行结果同样是保存失败，分析一下EF执行的SQL：

        exec sp_executesql N'update [dbo].[People] 
        set [SocialSecurityNumber] = @0
        where (([PersonId] = @1) and ([SocialSecurityNumber] = @2))
        ',N'@0 int,@1 int,@2 int',@0=123,@1=1,@2=12345678

可以看到，EF将我们要并发控制的列SocialSecurityNumber也作为一个筛选条件，这样firContext保存的时候也会因为的数据库中SocialSecurityNumber值变了，取不到对应的记录而更新失败。

补充一下：如果是EDMX如何将字段设置为Concurrency。很简单，在对应的字段上右键-属性。在打开的属性窗口中有一个并发模式，你将它选择为Fixed即可。
 

### 解决冲突
#### 数据库优先，使用Reload来解决乐观并发异常
Reload方法用数据库中的值来覆写entity的当前值。entity将会以某种形式会给用户，让他们再次进行改动，并重新保存，例如：

{% highlight java %}

using (var context = new BloggingContext()) 
{ 
    var blog = context.Blogs.Find(1); 
    blog.Name = "The New ADO.NET Blog"; 
 
    bool saveFailed; 
    do 
    { 
        saveFailed = false; 
 
        try 
        { 
            context.SaveChanges(); 
        } 
        catch (DbUpdateConcurrencyException ex) 
        { 
            saveFailed = true; 
 
            // Update the values of the entity that failed to save from the store 
            ex.Entries.Single().Reload(); 
        } 
 
    } while (saveFailed); 
}

{% endhighlight %}

模拟并发异常的一个好的方法是在SaveChanges上设置一个断点，然后使用其他数据库工具修改一个entity并保存。你也可以在SaveChanges之前插入一行直接更新数据库的SqlCommand，例如：

        context.Database.SqlCommand("UPDATE dbo.Blogs SET Name = 'Another Name' WHERE BlogId = 1");

DbUpdateConcurrencyException上的Entries方法返回更新失败的entities的DbEntityEntry的实例。这个方法当前对于并发问题是返回单一的值，它也可能返回多个更新异常的值。一个可选的方案就是获取所有的entities，并为他们每一个都执行Reload

#### 以用户优先来解决乐观并发
上面的例子中使用数据库优先，或是称为存储优先的Reload方法，因为在entity中的值被来自于数据的值覆盖。有些时候你可能希望的行为正好相反，使用当前entity中的值来覆盖数据库中的值。这称为用户优先的方式是通过获取当前数据库中的值，并将他们设为entity的original值来实现的

{% highlight java %}

using (var context = new BloggingContext()) 
{ 
    var blog = context.Blogs.Find(1); 
    blog.Name = "The New ADO.NET Blog"; 
 
    bool saveFailed; 
    do 
    { 
        saveFailed = false; 
        try 
        { 
            context.SaveChanges(); 
        } 
        catch (DbUpdateConcurrencyException ex) 
        { 
            saveFailed = true; 
 
            // Update original values from the database 
            var entry = ex.Entries.Single(); 
            entry.OriginalValues.SetValues(entry.GetDatabaseValues()); 
        } 
 
    } while (saveFailed); 
}

{% endhighlight %}

#### 自定义乐观并发异常解决方案
有时候你希望混合当前数据库和当前entity的值。这通常需要一些自定义的逻辑或是界面，例如，你可能要使用表单来呈现包含当前值，数据库值，最终值给用户。用户需要编辑最终值，并将最终值存入数据库。这个步骤可以在entity上使用GetDatabaseValues或是CurrentValues返回的DbPropertyValues对象来完成。例如：

{% highlight java %}

using (var context = new BloggingContext()) 
{ 
    var blog = context.Blogs.Find(1); 
    blog.Name = "The New ADO.NET Blog"; 
 
    bool saveFailed; 
    do 
    { 
        saveFailed = false; 
        try 
        { 
            context.SaveChanges(); 
        } 
        catch (DbUpdateConcurrencyException ex) 
        { 
            saveFailed = true; 
 
            // Get the current entity values and the values in the database 
            var entry = ex.Entries.Single(); 
            var currentValues = entry.CurrentValues; 
            var databaseValues = entry.GetDatabaseValues(); 
 
            // Choose an initial set of resolved values. In this case we 
            // make the default be the values currently in the database. 
            var resolvedValues = databaseValues.Clone(); 
 
            // Have the user choose what the resolved values should be 
            HaveUserResolveConcurrency(currentValues, databaseValues, resolvedValues); 
 
            // Update the original values with the database values and 
            // the current values with whatever the user choose. 
            entry.OriginalValues.SetValues(databaseValues); 
            entry.CurrentValues.SetValues(resolvedValues); 
        } 
    } while (saveFailed); 
} 
 
public void HaveUserResolveConcurrency(DbPropertyValues currentValues, 
                                       DbPropertyValues databaseValues, 
                                       DbPropertyValues resolvedValues) 
{ 
    // Show the current, database, and resolved values to the user and have 
    // them edit the resolved values to get the correct resolution. 
}

{% endhighlight %}

#### 使用object来自定义乐观并发异常解决方案
上述代码使用DbPropertyValues 实例来传递当前值，数据库值，和最终值。更方便的方法是使用你的entity实例类型来完成。使用DbPropertyValues的ToObject和SetValues来完成，比如

{% highlight java %}

using (var context = new BloggingContext()) 
{ 
    var blog = context.Blogs.Find(1); 
    blog.Name = "The New ADO.NET Blog"; 
 
    bool saveFailed; 
    do 
    { 
        saveFailed = false; 
        try 
        { 
            context.SaveChanges(); 
        } 
        catch (DbUpdateConcurrencyException ex) 
        { 
            saveFailed = true; 
 
            // Get the current entity values and the values in the database 
            // as instances of the entity type 
            var entry = ex.Entries.Single(); 
            var databaseValues = entry.GetDatabaseValues(); 
            var databaseValuesAsBlog = (Blog)databaseValues.ToObject(); 
 
            // Choose an initial set of resolved values. In this case we 
            // make the default be the values currently in the database. 
            var resolvedValuesAsBlog = (Blog)databaseValues.ToObject(); 
 
            // Have the user choose what the resolved values should be 
            HaveUserResolveConcurrency((Blog)entry.Entity, 
                                       databaseValuesAsBlog, 
                                       resolvedValuesAsBlog); 
 
            // Update the original values with the database values and 
            // the current values with whatever the user choose. 
            entry.OriginalValues.SetValues(databaseValues); 
            entry.CurrentValues.SetValues(resolvedValuesAsBlog); 
        } 
 
    } while (saveFailed); 
} 
 
public void HaveUserResolveConcurrency(Blog entity, 
                                       Blog databaseValues, 
                                       Blog resolvedValues) 
{ 
    // Show the current, database, and resolved values to the user and have 
    // them update the resolved values to get the correct resolution. 
}

{% endhighlight %}
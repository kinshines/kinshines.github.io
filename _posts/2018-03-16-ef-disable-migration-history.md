---
layout: post
title: EF 禁用对表__MigrationHistory的查询
author: kinshines
date:   2018-03-16
categories: ef
permalink: /archivers/ef-disable-migration-history
---

<p class="lead">使用了 EF 的Code First会在第一次EF查询的时候会对__MigrationHistory访问，是为了检查数据库和model是否匹配，以保证ef能正常运行。通过监测会先执行下面的sql：</p>

{% highlight sql %}

SELECT 
[GroupBy1].[A1] AS [C1]
FROM ( SELECT 
    COUNT(1) AS [A1]
    FROM [dbo].[__MigrationHistory] AS [Extent1]
)  AS [GroupBy1]
GO
 
SELECT TOP (1) 
[Extent1].[Id] AS [Id], 
[Extent1].[ModelHash] AS [ModelHash]
FROM [dbo].[EdmMetadata] AS [Extent1]
ORDER BY [Extent1].[Id] DESC
GO

{% endhighlight %}

这段sql语句其实中只是在开发的时候有用，发布到生产环境，可以把这个给禁用了以提高性能。解决办法：

Application_Start加代码：

{% highlight java %}

Database.SetInitializer<DefaultDbContext>(null);

{% endhighlight %}
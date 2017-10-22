---
layout: post
title: SQL Server 分页查询方法汇总
author: kinshines
date:   2017-10-11
categories: db
permalink: /archivers/sql-server-page-query
---

<p class="lead">SQL Server 不像MySQL那样可以直接简单使用limit 进行分页查询，本文汇总了以下五种分页查询的方法，供参考。假设现有UserInfo表，主键：UserId。设参数：<br/>
PageSize = 30 <br/>
PageNumber = 21 </p>

### 方法一：(最常用的分页代码， top，not in)

        select top 30 UserId from UserInfo where UserId not in (select top 600 UserId from UserInfo order by UserId) order by UserId

备注： 注意前后的order by 一致

### 方法二：(not exists， not in 的另一种写法而已)

        select top 30 * from UserLog where not exists (select 1 from (select top 600 LogId from UserLog order by LogId) a where a.LogId = UserLog.LogId) order by LogId

备注：EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False。此处的 select 1 from 也可以是select 2 from，select LogId from, select * from 等等，不影响查询。而且select 1 效率最高，不用查字典表。效率值比较：1 > anycol > *

### 方法三：(top / max， 局限于使用可比较列排序的时候)

        select top 30 * from UserLog where LogId > (select max(LogId) from (select top 600 LogId from UserLog order by LogId) a ) order by LogId

备注：这里max()函数也可以用于文本列，文本列的比较会根据字母顺序排列，数字 < 字母（无视大小写） < 中文字符

### 方法四：(row_number() over (order by LogId))

        select top 30 * from ( select row_number() over (order by LogId) as rownumber,* from UserLog) a where rownumber > 600 order by LogId

        select * from (select row_number()over(order by LogId) as rownumber,* from UserLog) a
        where rownumber > 600 and rownumber < 630 order by LogId

        select * from (select row_number()over(order by LogId) as rownumber,* from UserLog) a
        where rownumber between 600 and  630 order by LogId

备注:  这里rownumber方法属于排名开窗函数（sum, min, avg等属于聚合开窗函数，ORACLE中叫分析函数)的一种，搭配over关键字使用。

### 方法五：(offset /fetch next, SQL Server 2012支持)

        select * from UserLog Order by LogId offset 600 rows fetch next 30 rows only

备注： 性能参考文章[《SQL Server 2012使用OFFSET/FETCH NEXT分页及性能测试》](http://www.cnblogs.com/hukn/archive/2012/11/29/sql_server_2012_paging_using_offset_fetch_next_performance_test.html)




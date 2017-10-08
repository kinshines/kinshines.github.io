---
layout: post
title: SQL-LINQ-Lambda语法对照
author: kinshines
date:   2016-04-22
categories: ef
permalink: /archivers/sql-linq-lambda
---

<p class="lead">本文将SQL、LINQ、Lambda 三种语法作一个横向对照，便于加深理解。</p>

<table class="table">
  <thead>
    <tr>
      <th>SQL</th>
      <th>LINQ</th>
      <th>Lambda</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SELECT * FROM HumanResources.Employee</td>
      <td>from e in Employees select e</td>
      <td>Employees .Select (e => e)</td>
    </tr>
    <tr>
      <td>SELECT e.LoginID, e.JobTitle
              FROM HumanResources.Employee AS e</td>
      <td>from e in Employees
              select new {e.LoginID, e.JobTitle}</td>
      <td>Employees.Select (
              e => new {
                  LoginID = e.LoginID, 
                  JobTitle = e.JobTitle
                  }
              )
   </td>
    </tr>
    <tr>
      <td>SELECT e.LoginID AS ID, e.JobTitle AS Title
              FROM HumanResources.Employee AS e</td>
      <td>from e in Employees
              select new {ID = e.LoginID, Title = e.JobTitle}</td>
      <td>
      Employees.Select (
      e => new 
         {
            ID = e.LoginID, 
            Title = e.JobTitle
         }
      )
      </td>
    </tr>
    <tr>
    <td>
    SELECT DISTINCT e.JobTitle
FROM HumanResources.Employee AS e
    </td>
    <td>
    (from e in Employees
select e.JobTitle).Distinct()
    </td>
    <td>
    Employees
   .Select (e => e.JobTitle)
   .Distinct ()
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.*
FROM HumanResources.Employee AS e
WHERE e.LoginID = 'test'
    </td>
    <td>
    from e in Employees
where e.LoginID == "test"
select e
    </td>
    <td>
    Employees
   .Where (e => (e.LoginID == "test"))
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.*
FROM HumanResources.Employee AS e
WHERE e.LoginID = 'test' AND e.SalariedFlag = 1
    </td>
    <td>
    from e in Employees
where e.LoginID == "test" && e.SalariedFlag
select e
    </td>
    <td>
    Employees
   .Where (e => ((e.LoginID == "test") && e.SalariedFlag))
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.* FROM HumanResources.Employee AS e
WHERE e.VacationHours >= 2 AND e.VacationHours <= 10
    </td>
    <td>
    from e in Employees
where e.VacationHours >= 2 && e.VacationHours <= 10
select e
    </td>
    <td>
    Employees
   .Where (e => (((Int32)(e.VacationHours) >= 2) && ((Int32)(e.VacationHours) <= 10)))
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.* FROM HumanResources.Employee AS e
ORDER BY e.NationalIDNumber
    </td>
    <td>
    from e in Employees
orderby e.NationalIDNumber
select e
    </td>
    <td>
    Employees
   .OrderBy (e => e.NationalIDNumber)
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.* FROM HumanResources.Employee AS e
ORDER BY e.HireDate DESC, e.NationalIDNumber
    </td>
    <td>
    from e in Employees
orderby e.HireDate descending, e.NationalIDNumber
select e
    </td>
    <td>
    Employees
   .OrderByDescending (e => e.HireDate)
   .ThenBy (e => e.NationalIDNumber)
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.* FROM HumanResources.Employee AS e
WHERE e.JobTitle LIKE 'Vice%' OR SUBSTRING(e.JobTitle, 0, 3) = 'Pro'
    </td>
    <td>
    from e in Employees
where e.JobTitle.StartsWith("Vice") || e.JobTitle.Substring(0, 3) == "Pro"
select e
    </td>
    <td>
    Employees
   .Where (e => (e.JobTitle.StartsWith ("Vice") || (e.JobTitle.Substring (0, 3) == "Pro")))
    </td>
    </tr>
    <tr>
    <td>
    SELECT SUM(e.VacationHours)
FROM HumanResources.Employee AS e
    </td>
    <td>
    </td>
    <td>
    Employees.Sum(e => e.VacationHours);
    </td>
    </tr>
    <tr>
    <td>
    SELECT COUNT(*) FROM HumanResources.Employee AS e
    </td>
    <td>
    </td>
    <td>
    Employees.Count();
    </td>
    </tr>
    <tr>
    <td>
    SELECT SUM(e.VacationHours) AS TotalVacations, e.JobTitle
FROM HumanResources.Employee AS e
GROUP BY e.JobTitle
    </td>
    <td>
    from e in Employees
group e by e.JobTitle into g
select new {JobTitle = g.Key, TotalVacations = g.Sum(e => e.VacationHours)}
    </td>
    <td>
    Employees
   .GroupBy (e => e.JobTitle)
   .Select (
      g => new 
         {
            JobTitle = g.Key, 
            TotalVacations = g.Sum (e => (Int32)(e.VacationHours))
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT e.JobTitle, SUM(e.VacationHours) AS TotalVacations
FROM HumanResources.Employee AS e
GROUP BY e.JobTitle
HAVING e.COUNT(*) > 2
    </td>
    <td>
    from e in Employees
group e by e.JobTitle into g
where g.Count() > 2
select new {JobTitle = g.Key, TotalVacations = g.Sum(e => e.VacationHours)}
    </td>
    <td>
    Employees
   .GroupBy (e => e.JobTitle)
   .Where (g => (g.Count () > 2))
   .Select (
      g => 
         new 
         {
            JobTitle = g.Key, 
            TotalVacations = g.Sum (e => (Int32)(e.VacationHours))
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT *
FROM Production.Product AS p, Production.ProductReview AS pr
    </td>
    <td>
    from p in Products
from pr in ProductReviews
select new {p, pr}
    </td>
    <td>
    Products
   .SelectMany (
      p => ProductReviews, 
      (p, pr) => 
         new 
         {
            p = p, 
            pr = pr
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT *
FROM Production.Product AS p
INNER JOIN Production.ProductReview AS pr ON p.ProductID = pr.ProductID
    </td>
    <td>
    from p in Products
join pr in ProductReviews on p.ProductID equals pr.ProductID
select new {p, pr}
    </td>
    <td>
    Products
   .Join (
      ProductReviews, 
      p => p.ProductID, 
      pr => pr.ProductID, 
      (p, pr) => 
         new 
         {
            p = p, 
            pr = pr
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT *
FROM Production.Product AS p
INNER JOIN Production.ProductCostHistory AS pch ON p.ProductID = pch.ProductID AND p.SellStartDate = pch.StartDate
    </td>
    <td>
    from p in Products
join pch in ProductCostHistories on new {p.ProductID, StartDate = p.SellStartDate} equals new {pch.ProductID, StartDate = pch.StartDate}
select new {p, pch}
    </td>
    <td>
    Products
   .Join (
      ProductCostHistories, 
      p => 
         new 
         {
            ProductID = p.ProductID, 
            StartDate = p.SellStartDate
         }, 
      pch => 
         new 
         {
            ProductID = pch.ProductID, 
            StartDate = pch.StartDate
         }, 
      (p, pch) => 
         new 
         {
            p = p, 
            pch = pch
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT *
FROM Production.Product AS p
LEFT OUTER JOIN Production.ProductReview AS pr ON p.ProductID = pr.ProductID
    </td>
    <td>
    from p in Products
join pr in ProductReviews on p.ProductID equals pr.ProductID
into prodrev
select new {p, prodrev}
    </td>
    <td>
    Products
   .GroupJoin (
      ProductReviews, 
      p => p.ProductID, 
      pr => pr.ProductID, 
      (p, prodrev) => 
         new 
         {
            p = p, 
            prodrev = prodrev
         }
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT p.ProductID AS ID
FROM Production.Product AS p
UNION
SELECT pr.ProductReviewID
FROM Production.ProductReview AS pr
    </td>
    <td>
    (from p in Products
select new {ID = p.ProductID}).Union(
from pr in ProductReviews
select new {ID = pr.ProductReviewID})
    </td>
    <td>
    Products
   .Select (
      p => 
         new 
         {
            ID = p.ProductID
         }
   )
   .Union (
      ProductReviews
         .Select (
            pr => 
               new 
               {
                  ID = pr.ProductReviewID
               }
         )
   )
    </td>
    </tr>
    <tr>
    <td>
    SELECT TOP (10) *
FROM Production.Product AS p
WHERE p.StandardCost < 100
    </td>
    <td>
    (from p in Products
where p.StandardCost < 100
select p).Take(10)
    </td>
    <td>
    Products
   .Where (p => (p.StandardCost < 100))
   .Take (10)
    </td>
    </tr>
    <tr>
    <td>
    SELECT *
FROM [Production].[Product] AS p
WHERE p.ProductID IN(
    SELECT pr.ProductID
    FROM [Production].[ProductReview] AS [pr]
    WHERE pr.[Rating] = 5
    )
    </td>
    <td>
    from p in Products
where (from pr in ProductReviews
where pr.Rating == 5
select pr.ProductID).Contains(p.ProductID)
select p
    </td>
    <td>
    Products
   .Where (
      p => 
         ProductReviews
            .Where (pr => (pr.Rating == 5))
            .Select (pr => pr.ProductID)
            .Contains (p.ProductID)
   )
    </td>
    </tr>
  </tbody>
</table>

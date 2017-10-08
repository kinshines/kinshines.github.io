---
layout: post
title: SQL-LINQ-Lambda语法对照
author: kinshines
date:   2016-04-22
categories: ef
permalink: /archivers/sql-linq-lambda
---

<p class="lead">本文将SQL、LINQ、Lambda 三种语法作一个横向对照，便于加深理解。</p>

<table class="table table-bordered">
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
  </tbody>
</table>

---
layout: post
title: MVC中DropDownList 和DropDownListFor的常用方法
author: kinshines
date:   2016-10-22
categories: mvc
permalink: /archivers/mvc-dropdown-list
---

<p class="lead">本文简单总结一下Mvc中DropDownList 和DropDownListFor的常用方法</p>

### 一、 非强类型：
{% highlight java %}
//Controller:
ViewData["AreId"] = from a in rp.GetArea()
                               select new SelectListItem { 
                               Text=a.AreaName,
                               Value=a.AreaId.ToString()
                               };
//View:
@Html.DropDownList("AreId")

//还可以给其加上一个默认选项：
@Html.DropDownList("AreId", "请选择");
{% endhighlight %}

### 二、 强类型：
DropDownListFor常用的是两个参数的重载，第一参数是生成的select的名称，第二个参数是数据，用于将绑定数据源至DropDownListFor

{% highlight java %}
//Model:

   public class SettingsViewModel
   {
       Repository rp =new Repository();
       public string ListName { get; set; }  
       public  IEnumerable<SelectListItem> GetSelectList()
       {
               var selectList = rp.GetArea().Select(a => new SelectListItem { 
                               Text=a.AreaName,
                               Value=a.AreaId.ToString()
                               });
               return selectList;
           }
       }

//Controller:
       public ActionResult Index()
       {
           return View(new SettingsViewModel());
       }

//View:
@model Applicationtest.Models.SettingsViewModel
@Html.DropDownListFor(m=>m.ListName,Model.GetSelectList(),"请选择")

{% endhighlight %}
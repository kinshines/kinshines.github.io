---
layout: post
title: 使用X-editable实现单项实时编辑保存
author: kinshines
date:   2016-06-07
categories: js
permalink: /archivers/js-x-editable
---

<p class="lead">Web开发中，通常使用表单编辑模式是将所有可编辑的项全部编辑好之后，点击保存按钮后统一保存，但在某些情景中，要求在展现表单某一项时即可切换至编辑模式，编辑完成后通过ajax保存该项，一款 js 插件<a href='https://github.com/vitalets/x-editable'>X-editable</a>应运而生，下面是在MVC5 项目中的应用示例，更多用法，参见<a href="http://vitalets.github.io/x-editable/">官方文档</a></p>

### 下载安装
使用VS创建一个基本的MVC5项目，Nuget安装x-editable

        Install-Package x-editable

涉及到编辑日期和时间的话，需要安装Moment.js

        Install-Package Moment.js

### 引用
_Layout模板页添加css，js引用
{% highlight html %}
        <link href="~/Content/bootstrap3-editable/css/bootstrap-editable.css" rel="stylesheet" />
        ...
        <script src="~/Scripts/bootstrap3-editable/js/bootstrap-editable.min.js"></script>
        <script src="~/Scripts/moment.min.js"></script>
{% endhighlight %}

### Server
HomeController修改如下：
{% highlight java %}
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }

        [HttpPost]
        public ActionResult Edit(string name,string pk,string value)
        {
            if (value.Length > 10)
            {
                return Json(new XeditResponse("name too long"));
            }
            //save db operation
            return Json(new XeditResponse());
        }

        public ActionResult SelectSource()
        {
            List<XeditDropDownItem> list = new List<XeditDropDownItem>()
            {
                new XeditDropDownItem("1","Active"),
                new XeditDropDownItem("2","Blocked"),
                new XeditDropDownItem("3","Deleted")
            };
            return Json(list, JsonRequestBehavior.AllowGet);
        }
    }
{% endhighlight %}

在Models文件夹下新建两个Model：XeditDropDownItem.cs和XeditResponse.cs
{% highlight java %}
    public class XeditDropDownItem
    {
        public XeditDropDownItem(string valueParam,string textParam)
        {
            value = valueParam;
            text = textParam;
        }
        public string value { get; set; }
        public string text { get; set; }
    }

    public class XeditResponse
    {
        public XeditResponse()
        {
            status = "ok";
        }
        public XeditResponse(string errorMsg)
        {
            status = "error";
            msg = errorMsg;
        }
        public string status { get; set; }
        public string msg { get; set; }
    }
{% endhighlight %}
### 应用
~/View/Home/Index.cshtml修改如下：
{% highlight html %}
@{
    ViewBag.Title = "Home Page";
}
<h2>X-editable Demo</h2>
<h3>text</h3>
<div class="row">
    <div class="col-md-3">
        <a href="#" id="username" data-type="text" data-pk="1" data-url="/home/edit" data-title="Enter username">superuser</a>
    </div>
</div>
<h3>textarea</h3>
<div class="row">
    <div class="col-md-3">
        <a href="#" id="comments" data-type="textarea" data-pk="1">awesome comment!</a>
    </div>
</div>
<h3>select</h3>
<div class="row">
    <div class="col-md-3">
        <a href="#" id="status" data-type="select" data-pk="1" data-value="2"></a>
    </div>
</div>
<h3>date</h3>
<div class="row">
    <div class="col-md-3">
        <a href="#" id="date" data-pk="1"  data-value="2016-07-07" data-title="Select date"></a>
    </div>
</div>
<h3>time</h3>
<div class="row">
    <div class="col-md-3">
        <a href="#" id="time" data-pk="1" data-value="11:48" data-title="Select time"></a>
    </div>
</div>
<h3>datetime</h3>
<div class="row">
    <div class="col-md-6">
        <a href="#" id="datetime" data-pk="1" data-value="2016-07-07 11:48" data-title="Select datetime"></a>
    </div>
</div>

@section scripts{ 
    <script>
        $.fn.combodate.defaults.minYear = 2010;
        $.fn.combodate.defaults.maxYear = 2020;
        $.fn.combodate.defaults.firstItem = name;
        $(function () {
            $('#username').editable({
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });

            $('#comments').editable({
                url:'/home/edit',
                title: 'Enter comments',
                rows: 10,
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });

            $('#status').editable({
                url: '/home/edit',
                title:'Select status',
                source: '/home/selectsource'
            });

            $('#date').editable({
                type:'combodate',
                url: '/home/edit',
                format: 'YYYY-MM-DD',
                template: 'YYYY-MM-DD',
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });
            $('#time').editable({
                type: 'combodate',
                url: '/home/edit',
                format: 'HH:mm',
                template: 'HH:mm',
                combodate: {
                    minuteStep: 1,
                },
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });

            $('#datetime').editable({
                type: 'combodate',
                format: 'YYYY-MM-DD HH:mm',
                template:'YYYY-MM-DD HH:mm',
                combodate: {
                    maxYear: 2017,
                    minuteStep: 1,
                },
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });
        });
    </script>
}
{% endhighlight %}

但是如果渲染的是一个table，想使每一行editable生效，就不能通过元素的id属性调用该方法，而应该使用data-name属性，如下：
{% highlight html %}
<a href="#" data-name="username" data-type="text" data-pk="1" data-url="/home/edit" data-title="Enter username">superuser</a>
<script>
$("a[data-name='username']").editable({
                success: function (response, newValue) {
                    if (response.status == 'error') return response.msg; //msg will be shown in editable form
                }
            });
</script>
{% endhighlight %}

### 效果
#### Demo
![Demo](https://kinshines.github.io/img/js-xeditable/x-editable_1.png)
#### text
![text](https://kinshines.github.io/img/js-xeditable/x-editable_2.png)
#### text with error
![text with error](https://kinshines.github.io/img/js-xeditable/x-editable_3.png)
#### textarea
![textarea](https://kinshines.github.io/img/js-xeditable/x-editable_4.png)
#### select
![select](https://kinshines.github.io/img/js-xeditable/x-editable_5.png)
#### datetime
![datetime](https://kinshines.github.io/img/js-xeditable/x-editable_6.png)






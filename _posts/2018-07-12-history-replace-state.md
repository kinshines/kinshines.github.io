---
layout: post
title: 利用HTML5的HISTORY.REPLACESTATE修改当前页面的URL
author: kinshines
date:   2018-07-12
categories: js
permalink: /archivers/history-replace-state
---

<p class="lead">我们知道浏览器有一个history对象，用来保存浏览历史，用户可以通过点击浏览器的后退或前进按钮在历史记录中切换。之前对history的操作的API主要是前进、后退、跳转等，而在HTML5中提供了2个新方法来加强对history的管理。</p>

        history.pushState(state, title, url);
        history.replaceState(state, title, url);

两个方法的参数是一样的。说明如下：
* state：一个与指定历史记录相关联的状态对象，当popstate事件触发时，会把该对象传入回调函数。如果不需要用到，可以传null
* title：页面的标题。但当前大多数浏览器都不支持或忽略这个值。可以传null
* url：添加或修改的history的网址。为了安全性，必须保持与当前URL同一个域。
顾名思义，两个方法的区别只是pushState添加一个最新的历史记录，而replaceState则是把当前的页面的历史记录替换掉。他们最大的特点是添加或替换历史记录后，浏览器地址栏会变成你传的地址，而页面并不会重新载入或跳转。

例如，假设当前的页面地址是example.com/1.html，且history中此时只有一条当前页面的记录。

当你执行 history.pushState(null, null, “2.html”)后，浏览器的地址栏会显示example.com/2.html，但并不会跳转到2.html，甚至并不会去检查2.html存不存在，只是加入一个最新的历史记录项。此时history中会有两个记录。假如你此时点击页面上的link跳转到另外一个页面后，点击浏览器的后退按钮，则url会变成example.com/2.html，如果此前的1.html的页面浏览器有缓存的话会显示1.html的内容，否则会发起请求example.com/2.html。如果再次点后退，url会变成example.com/1.html。

而如果执行 history.replaceState(null, null, “2.html”)的话，浏览器的地址栏也会显示example.com/2.html，区别是history中只有当前2.html的记录，而1.html的记录已被替换掉。

利用这些特性，可以用来修改当前页面的URL来达成某些目的，比如可以用来记住搜索条件。

如果页面是数据展示的页面，页面上有一些搜索或过滤的条件，当用户点击这些条件时，页面通过AJAX把结果呈现到页面上，当点击某个结果页面跳转到下一个页面后，用户点击浏览器的后退按钮回到前一个页面后，即使页面有缓存，AJAX获取的结果也不会保留在页面上。如果后退后，需要记住此前的搜索条件和结果，可以在用户点击搜索条件时，把搜索条件利用history.replaceState添加到页面的query string中。

{% highlight js %}

if (history.replaceState) {
    history.replaceState(condition, null, "?" + $.param(condition, true));
}

{% endhighlight %}

页面可以设置成no-cache，当用户后退时，浏览器会重新请求带搜索条件的URL，后台根据搜索条件，把对应的结果页面呈现出来，从而达到记住搜索条件和结果的目的。

## replaceState使用实例
在bootstrap-datepicker的[Demo网页](https://uxsolutions.github.io/bootstrap-datepicker/?markup=input&format=&weekStart=&startDate=&endDate=&startView=0&minViewMode=0&maxViewMode=4&todayBtn=false&clearBtn=false&language=en&orientation=auto&multidate=&multidateSeparator=&keyboardNavigation=on&forceParse=on#sandbox)上，就很巧妙的用history.replaceState实现对url的修改：

{% highlight js %}

function update_url(){
        if(history.replaceState){
                var query='?'+$('.sandbox-form').serialize();
                history.replaceState(null,null,query+'#sandbox');
        }
}

{% endhighlight %}

## 利用HISTORY.PUSHSTATE方法禁用浏览器的后退按钮
同样的，我们可以利用其中的API来达到禁用浏览器后退按钮的功能。

{% highlight js %}

if (window.history && window.history.pushState) {
    history.pushState("forward", null, location.href);
    $(window).on("popstate", function() {
         history.pushState("forward", null, location.href);
     });
}

{% endhighlight %}
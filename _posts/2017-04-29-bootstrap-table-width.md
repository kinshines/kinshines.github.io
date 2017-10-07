---
layout: post
title: bootstrap 控制table中列宽度
author: kinshines
date:   2017-04-29
categories: html
permalink: /archivers/bootstrap-table-column-width
---

<p class="lead">Bootstrap 中的table组件可以根据列中的内容自适用地匹配列宽度，然而当某一列内容过长时，反而会造成table不美观，因此这时我们希望能够显性地控制列宽度</p>

首先，可以在table元素上添加style
{% highlight html %}
 <table style="word-break:break-all; word-wrap:break-word;">
 </table>
{% endhighlight %}

第二步，在表头单元格th元素上设置width，可以是固定的px，也可以是百分比

{% highlight html %}
<tr>
<th style="width:100px"></th>
<th style="width:20%"></th>
</tr>
{% endhighlight %}
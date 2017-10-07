---
layout: post
title: jQuery函数attr()和prop()的区别
author: kinshines
date:   2017-03-25
categories: js
permalink: /archivers/jquery-attr-vs-prop
---

<p class="lead"> 在jQuery中，attr()函数和prop()函数都用于设置或获取指定的属性，它们的参数和用法也几乎完全相同。

但不得不说的是，这两个函数的用处却并不相同。下面我们来详细介绍这两个函数之间的区别。</p>

### 1. 操作对象不同
attr和prop分别是单词attribute和property的缩写，它们均表示"属性"的意思。

不过，在jQuery中，attribute和property却是两个不同的概念。attribute表示HTML文档节点的属性，property表示JS对象的属性。

{% highlight html %}
<!-- 这里的id、class、data_id均是该元素文档节点的attribute -->
<div id="message" class="test" data_id="123"></div>

<script type="text/javascript">
// 这里的name、age、url均是obj的property
var obj = { name: "CodePlayer", age: 18, url: "http://www.365mini.com/" };
</script>
{% endhighlight %}

在jQuery中，prop()函数的设计目标是用于设置或获取指定DOM元素(指的是JS对象，Element类型)上的属性(property)；attr()函数的设计目标是用于设置或获取指定DOM元素所对应的文档节点上的属性(attribute)。

### 2. 应用版本不同

attr()是jQuery 1.0版本就有的函数，prop()是jQuery 1.6版本新增的函数。毫无疑问，在1.6之前，只能使用attr()函数；1.6及以后版本，则可以根据实际需要选择对应的函数。

### 3. 用于设置的属性值类型不同
由于attr()函数操作的是文档节点的属性，因此设置的属性值只能是字符串类型，如果不是字符串类型，也会调用其toString()方法，将其转为字符串类型。

prop()函数操作的是JS对象的属性，因此设置的属性值可以为包括数组和对象在内的任意类型。

### 4. 其他细节问题
在jQuery 1.6之前，只有attr()函数可用，该函数不仅承担了attribute的设置和获取工作，还同时承担了property的设置和获取工作。例如：在jQuery 1.6之前，attr()也可以设置或获取tagName、className、nodeName、nodeType等DOM元素的property。

直到jQuery 1.6新增prop()函数，并用来承担property的设置或获取工作之后，attr()才只用来负责attribute的设置和获取工作。

此外，对于表单元素的checked、selected、disabled等属性，在jQuery 1.6之前，attr()获取这些属性的返回值为Boolean类型：如果被选中(或禁用)就返回true，否则返回false。

但是从1.6开始，使用attr()获取这些属性的返回值为String类型，如果被选中(或禁用)就返回checked、selected或disabled，否则(即元素节点没有该属性)返回undefined。并且，在某些版本中，这些属性值表示文档加载时的初始状态值，即使之后更改了这些元素的选中(或禁用)状态，对应的属性值也不会发生改变。

因为jQuery认为：attribute的checked、selected、disabled就是表示该属性初始状态的值，property的checked、selected、disabled才表示该属性实时状态的值(值为true或false)。

因此，在jQuery 1.6及以后版本中，请使用prop()函数来设置或获取checked、selected、disabled等属性。对于其它能够用prop()实现的操作，也尽量使用prop()函数。

{% highlight html %}
<input id="uid" type="checkbox" checked="checked" value="1">

<script type="text/javascript">
// 当前jQuery版本为1.11.1
var uid = document.getElementById("uid");
var $uid = $(uid);

document.writeln( $uid.attr("checked") ); // checked
document.writeln( $uid.prop("checked") ); // true

// 取消复选框uid的选中(将其设为false即可)
// 相当于 uid.checked = false;
$uid.prop("checked", false);

// attr()获取的是初始状态的值，即使取消了选中，也不会改变
document.writeln( $uid.attr("checked") ); // checked
// prop()获取的值已经发生变化
document.writeln( $uid.prop("checked") ); // false
</script>
{% endhighlight %}

### 5. 简单记法
前面说了那么多，还有迷糊怎么办，这里有一个简单的记法：
* 对于HTML元素本身就带有的固有属性，在处理时，使用prop方法
* 对于HTML元素我们自己自定义的DOM属性，在处理时，使用attr方法

举几个例子：

{% highlight html %}
<a href="http://www.baidu.com" target="_self" class="btn">百度</a>
<a href="#" id="link1" action="delete">删除</a>
{% endhighlight %}

第1个<a>元素的DOM属性有"href、target和class"，这些属性就是<a>元素本身就带有的属性，也是W3C标准里就包含有这几个属性，或者说在IDE里能够智能提示出的属性，这些就叫做固有属性。处理这些属性时，建议使用prop方法。

第2个<a>元素属性有"href、id和action"，很明显，前两个是固有属性，而后面一个"action"属性是我们自己自定义上去的，<a>元素本身是没有这个属性的。这种就是自定义的DOM属性。处理这些属性时，建议使用attr方法。使用prop方法取值和设置属性值时，都会返回undefined值

再举一个例子：

{% highlight html %}
<input id="chk1" type="checkbox" />是否可见
<input id="chk2" type="checkbox" checked="checked" />是否可见
{% endhighlight %}

像checkbox，radio和select这样的元素，选中属性对应“checked”和“selected”，这些也属于固有属性，因此需要使用prop方法去操作才能获得正确的结果。

{% highlight js %}
$("#chk1").prop("checked") == false
$("#chk2").prop("checked") == true
{% endhighlight %}

如果上面使用attr方法，则会出现：

{% highlight js %}
$("#chk1").attr("checked") == undefined
$("#chk2").attr("checked") == "checked"
{% endhighlight %}
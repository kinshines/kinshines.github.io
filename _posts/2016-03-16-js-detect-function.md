---
layout: post
title: JavaScript中判断函数、变量是否存在
author: kinshines
date:   2016-03-16
categories: js
permalink: /archivers/js-detect-function
---

<p class="lead">本文主要介绍了JavaScript中判断函数、变量是否存在，用于保证js兼容性的情形</p>

#### 是否存在指定函数 function

{% highlight js %}
    function isExitsFunction(funcName) {
      if (typeof(funcName) === "function") {
        return true;
      }
        return false;
      }
{% endhighlight %}

#### 类似PHP常用的判断函数是否存在,不存在则创建

{% highlight js %}
    if (typeof String.prototype.endsWith != 'function') {
      String.prototype.endsWith = function(suffix) {
        return this.indexOf(suffix, this.length - suffix.length) !== -1;
          };
      }
{% endhighlight %}

#### 判断js函数是否存在，如果存在则执行

{% highlight js %}
          try 
          { 
             if(typeof(funcName)==="function") 
              {
                   funcName();
               }
          }catch(e)
          {
            //alert("not function"); 
          } 
{% endhighlight %}

#### 是否存在指定变量 

{% highlight js %}
          function isExitsVariable(variableName) {
            try {
              if (typeof(variableName) == "undefined") {
                //alert("value is undefined"); 
                return false;
                } else {
              //alert("value is true"); 
              return true;
                }
           } catch(e) {}
              return false;
        }
{% endhighlight %}



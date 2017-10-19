---
layout: post
title: MVC中对Enum中的枚举项应用DisplayTemplates和EditorTemplates
author: kinshines
date:   2017-03-04
categories: mvc
permalink: /archivers/enum-display-editor-templates
---

<p class="lead">本文介绍Enum中的枚举项应用DisplayTemplates和EditorTemplates</p>

### DisplayTemplates
现有枚举项如下，首先给每一项通过Display属性声明显示名称
{% highlight java %}
public enum MyEnum : byte
    {
        [Display(Name = "Zéro")]
        Zero,

        [Display(Name = "Un")]
        One,

        [Display(Name = "Duex")]
        Two,

        [Display(Name = "Trois")]
        Three,

        [Display(Name = "Quatre")]
        Four,
    }
{% endhighlight %}

其次，在 Views/Shared/DisplayTemplates 文件夹下，新增Partial View，命名为：Enum.cshtml  内容如下：

{% highlight java %}
@model Enum

@if (EnumHelper.IsValidForEnumHelper(ViewData.ModelMetadata))
{
   // Display Enum using same names (from [Display] attributes) as in editors
   string displayName = null;
   foreach (SelectListItem item in EnumHelper.GetSelectList(ViewData.ModelMetadata, (Enum)Model))
   {
       if (item.Selected)
       {
           displayName = item.Text ?? item.Value;
       }
   }

   // Handle the unexpected case that nothing is selected
   if (String.IsNullOrEmpty(displayName))
   {
       if (Model == null)
       {
           displayName = String.Empty;
       }
       else
       {
           displayName = Model.ToString();
       }
   }
   
   @Html.DisplayTextFor(model => displayName)
}
else
{
   // This Enum type is not supported.  Fall back to the text.
   @Html.DisplayTextFor(model => model)
}
{% endhighlight %}

### EditorTemplates

#### Enum.cshtml  以select控件的方式加载

{% highlight java %}
@model Enum

@if (EnumHelper.IsValidForEnumHelper(ViewData.ModelMetadata))
{
   @Html.EnumDropDownListFor(model => model, htmlAttributes: new { @class = "form-control" })
}
else
{
   @Html.TextBoxFor(model => model, htmlAttributes: new { @class = "form-control" })
}
{% endhighlight %}

#### Enum-radio.cshtml  以radio-button控件的方式加载

{% highlight java %}
@model Enum

@if (EnumHelper.IsValidForEnumHelper(ViewData.ModelMetadata))
{
   // Provide radio buttons for current Enum value
   string name = ViewData.ModelMetadata.PropertyName;
   string id = Html.IdForModel().ToString();
   int itemNum = 0;
   
   foreach (SelectListItem item in EnumHelper.GetSelectList(ViewData.ModelMetadata, (Enum)Model))
   {
       string myId = id + itemNum++.ToString();
       string myChecked = item.Selected ? "checked" : null;

       <label class="radio-inline">
           <input type="radio" id="@(myId)" name="@(name)" value="@(item.Value)" checked="@(myChecked)" /> @(item.Text)
       </label>
   }
}
else
{
   // This Enum type is not supported.  Fall back to a text box.
   @Html.TextBoxFor(model => model)
}
{% endhighlight %}

参考源码：[ASP.NET-Source Code](https://aspnet.codeplex.com/SourceControl/latest#Samples/MVC/EnumSample/EnumSample/Views/Shared/DisplayTemplates/Enum.cshtml)
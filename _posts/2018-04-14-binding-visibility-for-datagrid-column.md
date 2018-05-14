---
layout: post
title: 通过绑定修改DataGrid的Column的Visibility
author: kinshines
date:   2018-04-14
categories: wpf
permalink: /archivers/binding-visibility-for-datagrid-column
---

<p class="lead">问题：因为DataGridColumns不是可视树的一部分，无法继承datagrid的datacontext，以下提供两种解决方案</p>

### 一、代理类
#### 定义Proxy类
{% highlight java %}

    public class BindingProxy: Freezable
    {
        #region Overrides of Freezable

        protected override Freezable CreateInstanceCore()
        {
            return new BindingProxy();
        }

        #endregion

        public object Data
        {
            get { return (object)GetValue(DataProperty); }
            set { SetValue(DataProperty, value); }
        }

        // Using a DependencyProperty as the backing store for Data.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty DataProperty =
            DependencyProperty.Register("Data", typeof(object), typeof(BindingProxy), new UIPropertyMetadata(null));
    }

{% endhighlight %}

#### View页面应用：
{% highlight xml %}

<DataGrid>  
        <DataGrid.Resources>  
            <local:BindingProxy x:Key="proxy" Data="{Binding}"></local:BindingProxy>  
        </DataGrid.Resources>  
        <DataGrid.Columns>  
        <DataGridTextColumn Header="Grade"  
            Visibility="{Binding Data.MyColumnVisibility, Source={StaticResource proxy}}"  
            Binding="{Binding Path=Grade}">  
        </DataGridTextColumn>  
    </DataGrid.Columns>  
</DataGrid>  

{% endhighlight %}

### 二、代元素
#### 在用户控件的Resources里加一个代理FrameworkElement，绑定控件的datacontex，并设置为不可见

{% highlight xml %}

<FrameworkElement x:Name="dummyElement" Visibility="Collapsed" DataContext="{Binding}"/>  

{% endhighlight %}

#### 将DataGridColumn的Visibility绑定Datacontext中的属性，比如IsEnable、或NoVisibility，Source采用x:Reference dummyElement,必要情况下，再添加Converter转换

{% highlight xml %}

<DataGrid>  
    <DataGrid.Columns>  
        <DataGridTextColumn Header="No."   
            Visibility="{Binding DataContext.IsEnable, Source={x:Reference dummyElement},Converter={StaticResource BooleanToVisibilityConverter}}">  
        </DataGridTextColumn>  
        <DataGridTextColumn Header="Name"  
            Visibility="{Binding DataContext.NoVisibility, Source={x:Reference dummyElement}}">  
        </DataGridTextColumn>  
    </DataGrid.Columns>  
</DataGrid>  
{% endhighlight %}
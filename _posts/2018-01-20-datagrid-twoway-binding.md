---
layout: post
title: DataGrid 控件模板列实现双向绑定(Binding)
author: kinshines
date:   2018-01-20
categories: wpf
permalink: /archivers/datagrid-twoway-binding
---

<p class="lead">在WPF中，使用控件DataGrid控件的模板列实现双向绑定时，UI的数据更新不到数据源，下面提供一种解决方案</p>

### xaml
现在将CheckBox控件作为模板列嵌套进DataGrid中，希望在UI界面中对CheckBox的勾选能更新至数据源，需要注意的是在绑定语句中一定要加上 UpdateSourceTrigger=PropertyChanged，否则更新不能生效：
{% highlight xml %}
    <StackPanel>
        <TextBlock x:Name="ShowBBValue" Text="{Binding BB}" />
        <DataGrid x:Name="dataGrid" Grid.Row="0" ItemsSource="{Binding bindingData}" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="AA" Binding="{Binding AA,Mode=TwoWay,UpdateSourceTrigger=PropertyChanged}" />
                <DataGridTemplateColumn Header="BB">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <CheckBox IsChecked="{Binding BB, Mode=TwoWay,UpdateSourceTrigger=PropertyChanged}" />
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </StackPanel>
{% endhighlight %}

### C\#
{% highlight java %}
            
    public partial class MainWindow : Window
    {
        class1 a = new class1();
        ObservableCollection<class1> bindingData = new ObservableCollection<class1>();
        public MainWindow()
        {
            InitializeComponent();
            InitDataBinding();
        }

        void InitDataBinding()
        {
            a.AA = "aaaaa";
            a.BB = true;
            bindingData.Add(a);
            ShowBBValue.DataContext = this.a;
            dataGrid.ItemsSource = bindingData;
        }

    }

    
 
    public class class1 : INotifyPropertyChanged  
    {  
   
        private bool  _bb;
        private string _aa;

        public string AA
        {
            get { return _aa; }
            set { this._aa= value; OnPropertyChanged("AA"); }
        }
        
        public bool BB
        { 
            get { return this._bb; } 
            set 
            { 
                this._bb= value; 
                OnPropertyChanged("BB"); 
            } 
        }  
        public event PropertyChangedEventHandler PropertyChanged;  
   
        void OnPropertyChanged(string name)  
        {  
            if (PropertyChanged != null)  
                this.PropertyChanged(this, new PropertyChangedEventArgs(name));  
        }  
   
    }  

{% endhighlight %}
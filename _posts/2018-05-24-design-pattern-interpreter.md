---
layout: post
title: 解释器模式
author: kinshines
date:   2018-05-24
categories: design-pattern
permalink: /archivers/design-pattern-interpreter
---

<p class="lead">
解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，它属于行为型模式。这种模式实现了一个表达式接口，该接口解释一个特定的上下文。
<br />
这种模式被用在 SQL 解析、符号处理引擎等。
</p>

### 意图
给定一个语言，定义它的文法表示，并定义一个解释器，这个解释器使用该标识来解释语言中的句子。
### 主要解决
对于一些固定文法构建一个解释句子的解释器。
### 何时使用
如果一种特定类型的问题发生的频率足够高，那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。这样就可以构建一个解释器，该解释器通过解释这些句子来解决该问题。
### 如何解决
构件语法树，定义终结符与非终结符。
### 关键代码
构件环境类，包含解释器之外的一些全局信息，一般是 HashMap。
### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {
            Context context = new Context();
            IList<AbstractExpression> list = new List<AbstractExpression>();
            list.Add(new TerminalExpression());
            list.Add(new NonterminalExpression());
            list.Add(new TerminalExpression());
            list.Add(new TerminalExpression());

            foreach (AbstractExpression exp in list)
            {
                exp.Interpret(context);
            }

            Console.Read();
        }
    }

    class Context
    {
        private string input;
        public string Input
        {
            get { return input; }
            set { input = value; }
        }

        private string output;
        public string Output
        {
            get { return output; }
            set { output = value; }
        }
    }

    abstract class AbstractExpression
    {
        public abstract void Interpret(Context context);
    }

    class TerminalExpression : AbstractExpression
    {
        public override void Interpret(Context context)
        {
            Console.WriteLine("终端解释器");
        }
    }

    class NonterminalExpression : AbstractExpression
    {
        public override void Interpret(Context context)
        {
            Console.WriteLine("非终端解释器");
        }
    }

{% endhighlight %}

### 示例
乐谱解释器

#### 步骤1 创建一个表达式接口

{% highlight java %}

      //表达式
    abstract class Expression
    {
        //解释器
        public void Interpret(PlayContext context)
        {
            if (context.PlayText.Length == 0)
            {
                return;
            }
            else
            {
                string playKey = context.PlayText.Substring(0, 1);
                context.PlayText = context.PlayText.Substring(2);
                double playValue = Convert.ToDouble(context.PlayText.Substring(0, context.PlayText.IndexOf(" ")));
                context.PlayText = context.PlayText.Substring(context.PlayText.IndexOf(" ") + 1);

                Excute(playKey, playValue);

            }
        }
        //执行
        public abstract void Excute(string key, double value);
    }

        //演奏内容
    class PlayContext
    {
        //演奏文本
        private string text;
        public string PlayText
        {
            get { return text; }
            set { text = value; }
        }
    }

{% endhighlight %}

#### 步骤2 创建实现了上述接口的实体类。

{% highlight java %}

     //音符
    class Note : Expression
    {
        public override void Excute(string key, double value)
        {
            string note = "";
            switch (key)
            {
                case "C":
                    note = "1";
                    break;
                case "D":
                    note = "2";
                    break;
                case "E":
                    note = "3";
                    break;
                case "F":
                    note = "4";
                    break;
                case "G":
                    note = "5";
                    break;
                case "A":
                    note = "6";
                    break;
                case "B":
                    note = "7";
                    break;

            }
            Console.Write("{0} ", note);
        }
    }

    //音阶
    class Scale : Expression
    {
        public override void Excute(string key, double value)
        {
            string scale = "";
            switch (Convert.ToInt32(value))
            {
                case 1:
                    scale = "低音";
                    break;
                case 2:
                    scale = "中音";
                    break;
                case 3:
                    scale = "高音";
                    break;

            }
            Console.Write("{0} ", scale);
        }
    }

    //音速
    class Speed : Expression
    {
        public override void Excute(string key, double value)
        {
            string speed;
            if (value < 500)
                speed = "快速";
            else if (value >= 1000)
                speed = "慢速";
            else
                speed = "中速";

            Console.Write("{0} ", speed);
        }
    }

{% endhighlight %}


#### 步骤3 使用解释器

{% highlight java %}

        static void Main(string[] args)
        {
            PlayContext context = new PlayContext();
            //音乐-上海滩
            Console.WriteLine("上海滩：");
            //context.演奏文本 = "T 500 O 2 E 0.5 G 0.5 A 3 E 0.5 G 0.5 D 3 E 0.5 G 0.5 A 0.5 O 3 C 1 O 2 A 0.5 G 1 C 0.5 E 0.5 D 3 D 0.5 E 0.5 G 3 D 0.5 E 0.5 O 1 A 3 A 0.5 O 2 C 0.5 D 1.5 E 0.5 D 0.5 O 1 B 0.5 A 0.5 O 2 C 0.5 O 1 G 3 P 0.5 O 2 E 0.5 G 0.5 A 3 E 0.5 G 0.5 D 3 E 0.5 G 0.5 A 0.5 O 3 C 1 O 2 A 0.5 G 1 C 0.5 E 0.5 D 3 D 0.5 E 0.5 G 3 D 0.5 E 0.5 O 1 A 3 A 0.5 O 2 C 0.5 D 1.5 E 0.5 D 0.5 O 1 B 0.5 A 0.5 G 0.5 O 2 C 3 P 0.5 O 3 C 0.5 C 0.5 O 2 A 0.5 O 3 C 2 P 0.5 O 2 A 0.5 O 3 C 0.5 O 2 A 0.5 G 2.5 G 0.5 E 0.5 A 1.5 G 0.5 C 1 D 0.25 C 0.25 D 0.5 E 2.5 E 0.5 E 0.5 D 0.5 E 2.5 O 3 C 0.5 C 0.5 O 2 B 0.5 A 3 E 0.5 E 0.5 D 1.5 E 0.5 O 3 C 0.5 O 2 B 0.5 A 0.5 E 0.5 G 2 P 0.5 O 2 E 0.5 G 0.5 A 3 E 0.5 G 0.5 D 3 E 0.5 G 0.5 A 0.5 O 3 C 1 O 2 A 0.5 G 1 C 0.5 E 0.5 D 3 D 0.5 E 0.5 G 3 D 0.5 E 0.5 O 1 A 3 A 0.5 O 2 C 0.5 D 1.5 E 0.5 D 0.5 O 1 B 0.5 A 0.5 G 0.5 O 2 C 3 ";
            context.PlayText = "T 500 O 2 E 0.5 G 0.5 A 3 E 0.5 G 0.5 D 3 E 0.5 G 0.5 A 0.5 O 3 C 1 O 2 A 0.5 G 1 C 0.5 E 0.5 D 3 ";
            //音乐-隐形的翅膀
            //Console.WriteLine("隐形的翅膀："); 
            //context.演奏文本 = "T 1000 O 1 G 0.5 O 2 C 0.5 E 1.5 G 0.5 E 1 D 0.5 C 0.5 C 0.5 C 0.5 C 0.5 O 1 A 0.25 G 0.25 G 1 G 0.5 O 2 C 0.5 E 1.5 G 0.5 G 0.5 G 0.5 A 0.5 G 0.5 G 0.5 D 0.25 E 0.25 D 0.5 C 0.25 D 0.25 D 1 A 0.5 G 0.5 E 1.5 G 0.5 G 0.5 G 0.5 A 0.5 G 0.5 E 0.5 D 0.5 C 0.5 C 0.25 D 0.25 O 1 A 1 G 0.5 A 0.5 O 2 C 1.5 D 0.25 E 0.25 D 1 E 0.5 C 0.5 C 3 O 1 G 0.5 O 2 C 0.5 E 1.5 G 0.5 E 1 D 0.5 C 0.5 C 0.5 C 0.5 C 0.5 O 1 A 0.25 G 0.25 G 1 G 0.5 O 2 C 0.5 E 1.5 G 0.5 G 0.5 G 0.5 A 0.5 G 0.5 G 0.5 D 0.25 E 0.25 D 0.5 C 0.25 D 0.25 D 1 A 0.5 G 0.5 E 1.5 G 0.5 G 0.5 G 0.5 A 0.5 G 0.5 E 0.5 D 0.5 C 0.5 C 0.25 D 0.25 O 1 A 1 G 0.5 A 0.5 O 2 C 1.5 D 0.25 E 0.25 D 1 E 0.5 C 0.5 C 3 E 0.5 G 0.5 O 3 C 1.5 O 2 B 0.25 O 3 C 0.25 O 2 B 1 A 0.5 G 0.5 A 0.5 O 3 C 0.5 O 2 E 0.5 D 0.5 C 1 C 0.5 C 0.5 C 0.5 O 3 C 1 O 2 G 0.25 A 0.25 G 0.5 D 0.25 E 0.25 D 0.5 C 0.25 D 0.25 D 3 E 0.5 G 0.5 O 3 C 1.5 O 2 B 0.25 O 3 C 0.25 O 2 B 1 A 0.5 G 0.5 A 0.5 O 3 C 0.5 O 2 E 0.5 D 0.5 C 1 C 0.5 C 0.5 C 0.5 O 3 C 1 O 2 G 0.25 A 0.25 G 0.5 D 0.25 E 0.25 D 0.5 C 0.5 C 3 ";
            Expression expression = null;
            try
            {
                while (context.PlayText.Length > 0)
                {
                    string str = context.PlayText.Substring(0, 1);
                    switch (str)
                    {
                        case "O":
                            expression = new Scale();
                            break;
                        case "T":
                            expression = new Speed();
                            break;
                        case "C":
                        case "D":
                        case "E":
                        case "F":
                        case "G":
                        case "A":
                        case "B":
                        case "P":
                            expression = new Note();
                            break;

                    }
                    expression.Interpret(context);

                }
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }

            Console.Read();
        }

{% endhighlight %}
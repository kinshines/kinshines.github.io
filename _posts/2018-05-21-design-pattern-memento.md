---
layout: post
title: 备忘录模式
author: kinshines
date:   2018-05-21
categories: design-pattern
permalink: /archivers/design-pattern-memento
---

<p class="lead">
备忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象。
<br/>
备忘录模式属于行为型模式。
</p>

### 意图
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。
### 主要解决
所谓备忘录模式就是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。
### 何时使用
很多时候我们总是需要记录一个对象的内部状态，这样做的目的就是为了允许用户取消不确定或者错误的操作，能够恢复到他原先的状态，使得他有"后悔药"可吃。

### 如何解决
通过一个备忘录类专门存储对象状态。
### 关键代码
客户不与备忘录类耦合，与备忘录管理类耦合。

### 基本代码

{% highlight java %}

    class Program
    {
        static void Main(string[] args)
        {

            Originator o = new Originator();
            o.State = "On";
            o.Show();

            Caretaker c = new Caretaker();
            c.Memento = o.CreateMemento();

            o.State = "Off";
            o.Show();

            o.SetMemento(c.Memento);
            o.Show();

            Console.Read();

        }
    }

    class Originator
    {
        private string state;
        public string State
        {
            get { return state; }
            set { state = value; }
        }

        public Memento CreateMemento()
        {
            return (new Memento(state));
        }

        public void SetMemento(Memento memento)
        {
            state = memento.State;
        }

        public void Show()
        {
            Console.WriteLine("State=" + state);
        }
    }

    class Memento
    {
        private string state;

        public Memento(string state)
        {
            this.state = state;
        }

        public string State
        {
            get { return state; }
        }
    }

    class Caretaker
    {
        private Memento memento;

        public Memento Memento
        {
            get { return memento; }
            set { memento = value; }
        }
    }

{% endhighlight %}

### 示例
打游戏存档

#### 步骤1 创建Memento类

{% highlight java %}

     //角色状态存储箱
    class RoleStateMemento
    {
        private int vit;
        private int atk;
        private int def;

        public RoleStateMemento(int vit, int atk, int def)
        {
            this.vit = vit;
            this.atk = atk;
            this.def = def;
        }

        //生命力
        public int Vitality
        {
            get { return vit; }
            set { vit = value; }
        }

        //攻击力
        public int Attack
        {
            get { return atk; }
            set { atk = value; }
        }

        //防御力
        public int Defense
        {
            get { return def; }
            set { def = value; }
        }
    }

{% endhighlight %}

#### 步骤2 创建Originator类。

{% highlight java %}

    class GameRole
    {
        //生命力
        private int vit;
        public int Vitality
        {
            get { return vit; }
            set { vit = value; }
        }

        //攻击力
        private int atk;
        public int Attack
        {
            get { return atk; }
            set { atk = value; }
        }

        //防御力
        private int def;
        public int Defense
        {
            get { return def; }
            set { def = value; }
        }

        //状态显示
        public void StateDisplay()
        {
            Console.WriteLine("角色当前状态：");
            Console.WriteLine("体力：{0}", this.vit);
            Console.WriteLine("攻击力：{0}", this.atk);
            Console.WriteLine("防御力：{0}", this.def);
            Console.WriteLine("");
        }

        //保存角色状态
        public RoleStateMemento SaveState()
        {
            return (new RoleStateMemento(vit, atk, def));
        }

        //恢复角色状态
        public void RecoveryState(RoleStateMemento memento)
        {
            this.vit = memento.Vitality;
            this.atk = memento.Attack;
            this.def = memento.Defense;
        }


        //获得初始状态
        public void GetInitState()
        {
            this.vit = 100;
            this.atk = 100;
            this.def = 100;
        }

        //战斗
        public void Fight()
        {
            this.vit = 0;
            this.atk = 0;
            this.def = 0;
        }
    }

{% endhighlight %}

#### 步骤3 创建CareTaker类。

{% highlight java %}

    //角色状态管理者
    class RoleStateCaretaker
    {
        private RoleStateMemento memento;

        public RoleStateMemento Memento
        {
            get { return memento; }
            set { memento = value; }
        }
    }

{% endhighlight %}

#### 步骤4 使用 CareTaker 和 Originator 对象

{% highlight java %}

    static void Main(string[] args)
        {

            //大战Boss前
            GameRole lixiaoyao = new GameRole();
            lixiaoyao.GetInitState();
            lixiaoyao.StateDisplay();

            //保存进度
            RoleStateCaretaker stateAdmin = new RoleStateCaretaker();
            stateAdmin.Memento = lixiaoyao.SaveState();

            //大战Boss时，损耗严重
            lixiaoyao.Fight();
            lixiaoyao.StateDisplay();

            //恢复之前状态
            lixiaoyao.RecoveryState(stateAdmin.Memento);

            lixiaoyao.StateDisplay();

            Console.Read();

        }

{% endhighlight %}
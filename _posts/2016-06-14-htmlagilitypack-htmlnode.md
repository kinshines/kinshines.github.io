---
layout: post
title: HtmlAgilityPack 之 HtmlNode类
author: kinshines
date:   2016-06-14
categories: c#
permalink: /archivers/HtmlAgilityPack-HtmlNode
---

<p class="lead">在抓取网页数据时，<a href="https://www.nuget.org/packages/HtmlAgilityPack">HtmlAgilityPack</a>是解析网页结构的利器，今天着重分析其HtmlNode类</p>

### 一、静态属性
{% highlight java %}
public static Dictionary<string, HtmlElementFlag> //ElementsFlags;获取集合的定义为特定的元素节点的特定行为的标志。表包含小写标记名称作为键和作为值的 HtmlElementFlags 组合 DictionaryEntry 列表。
public static readonly string HtmlNodeTypeNameComment;　　//获取一个注释节点的名称。实际上，它被定义为 '#comment
public static readonly string HtmlNodeTypeNameDocument;　  //获取文档节点的名称。实际上，它被定义为 '#document'
public static readonly string HtmlNodeTypeNameText;　　　　  //获取一个文本节点的名称。实际上，它被定义为 '#text'
{% endhighlight %}

### 二、属性
{% highlight java %}
Attributes 　　　　　　　　　　　　获取节点的属性集合 
ChildNodes　　　　　　　　　　　　获取子节点集合(包括文本节点) 
Closed　　　　　　　　　　　　　　该节点是否已关闭(</xxx>) 
ClosingAttributes　　　　　　　　  在关闭标签的属性集合 
FirstChild　　　　　　　　　　　　  获取第一个子节点 
HasAttributes　　　　　　　　　　  判断该节点是否含有属性 
HasChildNodes　　　　　　　　　　判断该节点是否含有子节点 
HasClosingAttributes　　　　　　  判断该节点的关闭标签是否含有属性(</xxx class="xxx">) 
Id　　　　　　　　　　　　　　　　 获取该节点的Id属性 
InnerHtml　　　　　　　　　　　　 获取该节点的Html代码 
InnerText　　　　　　　　　　　　 获取该节点的内容，与InnerHtml不同的地方在于它会过滤掉Html代码，而InnerHtml是连Html代码一起输出 
LastChild　　　　　　　　　　　　  获取最后一个子节点 
Line　　　　　　　　　　　　　　　 获取该节点的开始标签或开始代码位于整个HTML源代码的第几行(行号) 
LinePosition　　　　　　　　　　　 获取该节点位于第几列 
Name　　　　　　　　　　　　　　  Html元素名 
NextSibling　　　　　　　　　　　　获取下一个兄弟节点 
NodeType　　　　　　　　　　　　  获取该节点的节点类型 
OriginalName　　　　　　　　　　　获取原始的未经更改的元素名 
OuterHtml　　　　　　　　　　　　 整个节点的代码 
OwnerDocument　　　　　　　　　节点所在的HtmlDocument文档 
ParentNode　　　　　　　　　　　　获取该节点的父节点 
PreviousSibling　　　　　　　　　　获取前一个兄弟节点 
StreamPosition　　　　　　　　　　该节点位于整个Html文档的字符位置 
XPath　　　　　　　　　　　　　　  根据节点返回该节点的XPath
{% endhighlight %}

代码示例：

{% highlight java %}

static void Main(string[] args)
        {
            //<ul class="user_match clear">
            //    <li>年龄：21～30之间</li>
            //    <li>婚史：未婚</li>
            //    <li>地区：不限</li>
            //    <li>身高：175～185厘米之间</li>
            //    <li>学历：不限</li>
            //    <li>职业：不限</li>
            //    <li>月薪：不限</li>
            //    <li>住房：不限</li>
            //    <li>购车：不限</li>
            //</ul>


            WebClient wc = new WebClient();
            wc.BaseAddress = "http://www.juedui100.com/";
            wc.Encoding = Encoding.UTF8;
            HtmlDocument doc = new HtmlDocument();
            string html = wc.DownloadString("user/6971070.html");
            doc.LoadHtml(html);
            HtmlNode node = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[2]/ul[1]");     //根据XPath查找节点，跟XmlNode差不多
            Console.WriteLine(node.InnerText);  //输出节点内容      年龄：21～30之间 婚史：未婚 ......      与InnerHtml的区别在于，它不会输出HTML代码
            Console.WriteLine(node.InnerHtml);  //输出节点Html <li>年龄：21～30之间</li> <li>婚史：未婚</li> ....
            Console.WriteLine(node.Name);       //输出 ul    Html元素名 

            HtmlAttributeCollection attrs = node.Attributes;
            foreach(var item in attrs)
            {
                Console.WriteLine(item.Name + " : " + item.Value);    //输出 class ：user_match clear
            }

            HtmlNodeCollection CNodes = node.ChildNodes;    //所有的子节点
            foreach (HtmlNode item in CNodes)
            {
                Console.WriteLine(item.Name + "-" + item.InnerText);  //输出 li-年龄：21～30之间#text-   li-婚史：未婚#text-     .......  别忘了文本节点也算
            }

            Console.WriteLine(node.Closed);     //输出True    //当前的元素节点是否已封闭

            Console.WriteLine("================================");

            HtmlAttributeCollection attrs1 = node.ClosingAttributes;    //获取在结束标记的 HTML 属性的集合。  例如</ul class="">
            Console.WriteLine(attrs1.Count);    //输出0

            HtmlNode node1 = node.FirstChild;   //悲剧了ul的第一个节点是一个 \n 换行文本节点 第二个节点才到第一个li
            Console.WriteLine(node1.NodeType);  //输出Text 文本节点
            HtmlNode node3 = node.LastChild;    //同样最后一个节点一样是 \n 文本节点
            Console.WriteLine(node3.NodeType);  //输出Text 文本节点

            HtmlNode node2 = node.SelectSingleNode("child::li[1]");     //获取当前节点的第一个子li节点
            Console.WriteLine(node2.XPath);     //根据节点生成XPath表达式  /html/body/div[4]/div[1]/div[2]/ul[1]/li[1]   妈了个B，强大

            Console.WriteLine(node.HasAttributes);          //输出 True   判断节点是否含有属性
            Console.WriteLine(node.HasChildNodes);          //输出 True   判断节点是否含有子节点
            Console.WriteLine(node.HasClosingAttributes);   //False     判断节点结束标记是否含有属性
            
            Console.WriteLine(node.Line);           //输出 155  该节点开始标记位于页面代码的第几行
            Console.WriteLine(node.LinePosition);   //输出 1   该节点开始标记位于第几列2
            Console.WriteLine(node.NodeType);       //输出 Element   该节点类型 此处为元素节点            
            Console.WriteLine(node.OriginalName);   //输出 ul
            HtmlNode node4 = node.SelectSingleNode("child::li[1]");
            Console.WriteLine(node4.InnerText);     //输出 年龄：21～30之间
            HtmlNode node5 = node4.NextSibling.NextSibling;     //获取下一个兄弟元素 因为有一个换行符的文本节点，因此要两次，跳过换行那个文本节点
            Console.WriteLine(node5.InnerText);     //输出 婚史:未婚
            HtmlNode node6 = node5.PreviousSibling.PreviousSibling;     //同样两次以跳过换行文本节点
            Console.WriteLine(node6.InnerText);     //输出 年龄：21～30之间
            HtmlNode node7 = node6.ParentNode;      //获取父节点
            Console.WriteLine(node7.Name);          //输出 ul
            string str = node.OuterHtml;
            Console.WriteLine(str);     //输出整个ul代码class="user_match clear"><li>年龄：21～30之间</li>...</ul>
            Console.WriteLine(node.StreamPosition); //输出7331    获取此节点的流位置在文档中，相对于整个文档(Html页面源代码)的开始。
            HtmlDocument doc1 = node.OwnerDocument;
            doc1.Save(@"D:\123.html");

            HtmlNode node8 = doc.DocumentNode.SelectSingleNode("//*[@id=\"coll_add_aid59710701\"]");
            //<a id="coll_add_aid59710701" style="display:block" class="coll_fix needlogin" href="javascript:coll_add(5971070)">收藏</a>
            Console.WriteLine(node8.Id);    //输出 coll_add_aid59710701   获取Id属性的内容

            Console.ReadKey();
        }

{% endhighlight %}

### 三、方法
{% highlight java %}
IEnumerable<HtmlNode> Ancestors(); 　　　　　　　　　　　　　　返回此元素的所有上级节点的集合。 
IEnumerable<HtmlNode> Ancestors(string name); 　　　　 　　　  返回此元素参数名字匹配的所有上级节点的集合。 
IEnumerable<HtmlNode> AncestorsAndSelf(); 　　　　　　 　　　　返回此元素的所有上级节点和自身的集合。 
IEnumerable<HtmlNode> AncestorsAndSelf(string name); 　　　　 返回此元素的名字匹配的所有上级节点和自身的集合。 
HtmlNode AppendChild(HtmlNode newChild); 　　　　　　 　　　　  将参数元素追加到为调用元素的子元素(追加在最后) 
void AppendChildren(HtmlNodeCollection newChildren);　　　　　　 将参数集合中的元素追加为调用元素的子元素(追加在最后) 
HtmlNode PrependChild(HtmlNode newChild); 　　　　　　　　　　   将参数中的元素作为子元素，放在调用元素的最前面 
void PrependChildren(HtmlNodeCollection newChildren); 　　　　　　将参数集合中的所有元素作为子元素，放在调用元素前面 
static bool CanOverlapElement(string name); 　　　　　　　　　　　 确定是否可以保存重复的元素 
IEnumerable<HtmlAttribute> ChildAttributes(string name); 　　　　获取所有子元素的属性(参数名要与元素名匹配) 
HtmlNode Clone(); 　　　　　　　　　　　　　　　　　　　　　　　　 本节点克隆到一个新的节点 
HtmlNode CloneNode(bool deep); 　　　　　　　　　　　　　　　　　节点克隆到一个新的几点，参数确定是否连子元素一起克隆 
HtmlNode CloneNode(string newName); 　　　　　　　　　　　　　　克隆的同时更改元素名 HtmlNode 
CloneNode(string newName, bool deep);　　　　　　　　  克隆的同时更改元素名。参数确定是否连子元素一起克隆 
void CopyFrom(HtmlNode node); 　　　　　　　　　　　　　　　　　 创建重复的节点和其下的子树。 
void CopyFrom(HtmlNode node, bool deep); 　　　　　　　　　　　 创建节点的副本。 
XPathNavigator CreateNavigator(); 　　　　　　　　　　　　　　　　 返回的一个对于此文档的XPathNavigator          
static HtmlNode CreateNode(string html); 　　　　　　　　　　　　  静态方法，允许用字符串创建一个新节点 
XPathNavigator CreateRootNavigator(); 　　　　　　　　　　　　　　创建一个根路径的XPathNavigator  
IEnumerable<HtmlNode> DescendantNodes(); 　　　　　　　　　　 获取所有子代节点 
IEnumerable<HtmlNode> DescendantNodesAndSelf(); 　　　　　　 获取所有的子代节点以及自身 
IEnumerable<HtmlNode> Descendants(); 　　　　　　　　　　　　　获取枚举列表中的所有子代节点 
IEnumerable<HtmlNode> Descendants(string name); 　　　　　　　获取枚举列表中的所有子代节点，注意元素名要与参数匹配 
IEnumerable<HtmlNode> DescendantsAndSelf(); 　　　　　　　　　获取枚举列表中的所有子代节点以及自身 
IEnumerable<HtmlNode> DescendantsAndSelf(string name);　　　 获取枚举列表中的所有子代节点以及自身，注意元素名要与参数匹配 
HtmlNode Element(string name); 　　　　　　　　　　　　　　　　　 根据参数名获取一个元素 
IEnumerable<HtmlNode> Elements(string name); 　　　　　　　　　根据参数名获取匹配的元素集合 
bool GetAttributeValue(string name, bool def); 　　　　　　　　　　 帮助方法，用来获取此节点的属性的值(布尔类型)。如果未找到该属性，则将返回默认值。 
int GetAttributeValue(string name, int def); 　　　　　　　　　　　　 帮助方法，用来获取此节点的属性的值(整型)。如果未找到该属性，则将返回默认值。 
string GetAttributeValue(string name, string def); 　　　　　　　　　帮助方法，用来获取此节点的属性的值(字符串类型)。如果未找到该属性，则将返回默认值。 
HtmlNode InsertAfter(HtmlNode newChild, HtmlNode refChild); 　　  将一个节点插入到第二个参数节点的后面，与第二个参数是兄弟关系 
HtmlNode InsertBefore(HtmlNode newChild, HtmlNode refChild); 　　讲一个节点插入到第二个参数节点的后面，与第二个参数是兄弟关系 
static bool IsCDataElement(string name); 　　　　　　　　　　　　　 确定是否一个元素节点是一个 CDATA 元素节点。 
static bool IsClosedElement(string name); 　　　　　　　　　　　　　确定是否封闭的元素节点 
static bool IsEmptyElement(string name); 　　　　　　　　　　　　   确定是否一个空的元素节点。 
static bool IsOverlappedClosingElement(string text); 　　　　　　　  确定是否文本对应于一个节点可以保留重叠的结束标记。 
void Remove(); 　　　　　　　　　　　　　　　　　　　　　　　　　　 从父集合中移除调用节点 
void RemoveAll(); 　　　　　　　　　　　　　　　　　　　　　　　　　 移除调用节点的所有子节点以及属性 
void RemoveAllChildren(); 　　　　　　　　　　　　　　　　　　　　　 移除调用节点的所有子节点 
HtmlNode RemoveChild(HtmlNode oldChild); 　　　　　　　　　　　　 移除调用节点的指定名字的子节点 
HtmlNode RemoveChild(HtmlNode oldChild, bool keepGrandChildren);移除调用节点调用名字的子节点，第二个参数确定是否连孙子节点一起移除 
HtmlNode ReplaceChild(HtmlNode newChild, HtmlNode oldChild);　　 将调用节点原有的一个子节点替换为一个新的节点，第二个参数是旧节点 
HtmlNodeCollection SelectNodes(string xpath);　　　　　　　　　　　根据XPath获取一个节点集合 
HtmlNode SelectSingleNode(string xpath); 　　　　　　　　　　　　　根据XPath获取唯一的一个节点 
HtmlAttribute SetAttributeValue(string name, string value); 　　　　 设置调用节点的属性 
string WriteContentTo(); 　　　　　　　　　　　　　　　　　　　　　  将该节点的所有子级都保存到一个字符串中。 
void WriteContentTo(TextWriter outText); 　　　　　　　　　　　　　将该节点的所有子级都保存到指定的 TextWriter。 
string WriteTo(); 　　　　　　　　　　　　　　　　　　　　　　　　　　将当前节点保存到一个字符串中。 
void WriteTo(TextWriter outText); 　　　　　　　　　　　　　　　　　将当前节点保存到指定的 TextWriter。 
void WriteTo(XmlWriter writer); 　　　　　　　　　　　　　　　　　　  将当前节点保存到指定的则 XmlWriter。
{% endhighlight %}

示例代码：

{% highlight java %}
static void Main(string[] args)
        {
            //<ul class="user_match clear">
            //    <li>年龄：21～30之间</li>
            //    <li>婚史：未婚</li>
            //    <li>地区：不限</li>
            //    <li>身高：175～185厘米之间</li>
            //    <li>学历：不限</li>
            //    <li>职业：不限</li>
            //    <li>月薪：不限</li>
            //    <li>住房：不限</li>
            //    <li>购车：不限</li>
            //</ul>


            WebClient wc = new WebClient();
            wc.BaseAddress = "http://www.juedui100.com/";
            wc.Encoding = Encoding.UTF8;
            HtmlDocument doc = new HtmlDocument();
            string html = wc.DownloadString("user/6971070.html");
            doc.LoadHtml(html);
            HtmlNode node = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[2]/ul[1]");     //根据XPath查找节点，跟XmlNode差不多

            IEnumerable<HtmlNode> nodeList = node.Ancestors();  //获取该元素所有的父节点的集合
            foreach (HtmlNode item in nodeList)
            {
                Console.Write(item.Name + " ");   //输出 div div body html #document
            }
            Console.WriteLine();

            IEnumerable<HtmlNode> nodeList1 = node.Ancestors("body");  //获取名字匹配的该元素的父集合,其实参数就是一个筛选的功能
            foreach (HtmlNode item in nodeList1)
            {
                Console.Write(item.Name + " ");   //输出 body
            }
            Console.WriteLine();

            IEnumerable<HtmlNode> nodeList2 = node.AncestorsAndSelf();  //获取所有的父节点和自身
            foreach (HtmlNode item in nodeList2)
            {
                Console.Write(item.Name + " "); //输出 ul div div div body html #document
            }
            Console.WriteLine();

            IEnumerable<HtmlNode> nodeList3 = node.AncestorsAndSelf("div");     //获取父节点和自身，参数用于筛选
            foreach (HtmlNode item in nodeList3)
            {
                Console.Write(item.Name + " "); //输出 div div div
            }
            Console.WriteLine();

            HtmlNode node1 = doc.CreateElement("li");
            node1.InnerHtml = "我是附加的li元素";
            node.AppendChild(node1);    //...<li>购车：不限</li> 后面加了一个<li>我是附加的li元素</li>
            Console.WriteLine(node.InnerHtml);


            HtmlNode node2 = doc.CreateElement("li");
            node2.InnerHtml = "新li一";
            HtmlNode node3 = doc.CreateElement("li");
            node3.InnerHtml = "新li二";
            HtmlNodeCollection nc = new HtmlNodeCollection(node2);
            nc.Add(node2);
            nc.Add(node3);
            node.AppendChildren(nc);    //一次过追加多个元素
            Console.WriteLine(node.InnerHtml);      //...<li>我是附加的li元素</li><li>新li一</li><li>新li二</li>

            Console.WriteLine(HtmlNode.CanOverlapElement("node2"));     //输出False   确定是否可以保存一个重复的元素

            IEnumerable<HtmlAttribute> attrs = node.ChildAttributes("class");   //获取子节点与自身的所有名为class的属性集合
            foreach (HtmlAttribute attr in attrs)
            {
                Console.Write(attr.Value);      //输出 user_match clear 
            }

            HtmlNode node4 = node.Clone();
            Console.WriteLine(node4.InnerHtml);     //输出node的代码，node已被复制到了node

            HtmlNode node5 = node.CloneNode(false); //参数决定是否复制子节点，与XmlNode一样
            Console.WriteLine(node5.OuterHtml);     //<ul class="user_match clear"></ul>    因为参数设为了false子节点没有被复制

            HtmlNode node6 = node.CloneNode("div");    //复制节点的同时，更改名字
            Console.WriteLine(node6.OuterHtml);        //输出 <div class="user_match clear"><li>年龄：21～30之间</li>...</div>  ul已被改为了div

            HtmlNode node7 = node.CloneNode("table",false);
            Console.WriteLine(node7.OuterHtml);        //输出<table class="user_match clear"></table>     参数为false所以没有复制子节点

            HtmlNode node8 = node.SelectSingleNode("child::li[1]");
            node.CopyFrom(node);
            Console.WriteLine(node.OuterHtml);
            Console.WriteLine("========================");
            //public void CopyFrom(HtmlNode node);
            //public void CopyFrom(HtmlNode node, bool deep);
            //public XPathNavigator CreateNavigator();
            //public XPathNavigator CreateRootNavigator();

            HtmlNode node9 = HtmlNode.CreateNode("<li>新节点</li>");   //直接用字符串创建节点，还是挺好用的
            Console.WriteLine(node9.OuterHtml);     //输出 <li>新节点</li>

            IEnumerable<HtmlNode> nodeList4 = node.DescendantNodes();   //获取所有的子节点集合
            foreach (HtmlNode item in nodeList4)
            {
                Console.Write(item.OuterHtml);      //输出 node的每个子li节点
            }
            Console.WriteLine("===================");

            IEnumerable<HtmlNode> nodeList5 = node.DescendantNodesAndSelf();
            foreach (HtmlNode item in nodeList5)
            {
                Console.Write(item.OuterHtml);      //输出自身<ul>..包括子节点<li>...</li></ul> 再输出所有的子li节点
            }
            Console.WriteLine();

            IEnumerable<HtmlNode> nodeList6 = node.DescendantNodes();   //获取枚举列表中的所有子代节
            foreach (HtmlNode item in nodeList6)
            {
                Console.Write(item.InnerText);  //输出所有的li节点的内容
            }
            Console.WriteLine("---------------");

            IEnumerable<HtmlNode> nodeList7 = node.Descendants("li");   //获取所有的子后代元素    //文本节点不在此范围内
            foreach(HtmlNode item in nodeList7)
            {
                Console.Write(item.InnerText);   
            }

            IEnumerable<HtmlNode> nodeList8 = node.DescendantsAndSelf("ul");   //获取所有的子后代元素    //文本节点不在此范围内
            foreach (HtmlNode item in nodeList8)
            {
                Console.Write(item.Name);       //输出 ul 参数实际上只相当于过滤的作用
            }

            HtmlNode node10 = node.Element("li");   //获取第一个子节点名称匹配的元素
            Console.WriteLine(node10.InnerText);        //输出 年龄：年龄：21～30之间
            Console.WriteLine("----------------------------------------");

            IEnumerable<HtmlNode> nodeList9 = node.Elements("li");
            foreach (HtmlNode item in nodeList9)
            {
                Console.Write(item.InnerText);      //输出 所有的li节点内容
            }
            Console.WriteLine();

            //换一个新的，好像有点乱了
            HtmlNode newnode = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[3]");
            //<div class="col say">
            //    <h3>爱情独白</h3>
            //    <p>愿得一心人，白首不相离。我一直相信我的另一半就在茫茫人海中，有一天一定会与我相遇。</p>
            //</div>

            //bool b = newnode.GetAttributeValue("class", false);   //获取一个布尔值的属性,没有找到则返回第二个参数的默认值
            //Console.WriteLine(b);
            //int i = newnode.GetAttributeValue("class", 0);        //获取一个整形的属性，没有找到则返回第二个参数的默认值
            //Console.WriteLine(i);

            string str = newnode.GetAttributeValue("class", "");    //获取一个字符串属性
            Console.WriteLine(str); //输出 col say

            HtmlNode node11 = HtmlNode.CreateNode("<b>我是加粗节点</b>");
            HtmlNode node12 = newnode.SelectSingleNode("h3");
            newnode.InsertAfter(node11, node12);    //意思是在node12代表的h3节点后面插入node11节点
            Console.WriteLine(newnode.InnerHtml);   //h3>爱情独白</h3><b>我是加粗节点</b><p>愿得一心人...      留意到b节点已经被插入到h3后面

            newnode.InsertBefore(node11, node12);   //再插入多一次，方法不同罢了，这次是在node12带包的h3前面插入
            Console.WriteLine(newnode.InnerHtml);   //<b>我是加粗节点</b><h3>爱情独白</h3><b>我是加粗节点</b><p>愿得一心人

            Console.WriteLine("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");
            newnode.RemoveChild(node11);    //移除了第一个<b>我是加粗节点</b>   此方法的重载，第二个参数决定是否移除孙子节点
            Console.WriteLine(newnode.InnerHtml);   //<h3>爱情独白</h3><b>我是加粗节点</b><p>愿得一心人....

            newnode.RemoveAllChildren();        //移除所有子节点
            Console.WriteLine(newnode.OuterHtml);   //<div class="col say"></div>   所有子节点都被移除了

            newnode.RemoveAll();                    //移除所有的属性和子节点，由于子节点已经被上个方法移除了，因此这次连属性也移除了
            Console.WriteLine(newnode.OuterHtml);   //输出 <div></div>    注意到属性也被移除了。

            //都移除光了，再来一个，还是刚才那个
            HtmlNode newnode1 = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[3]");
            Console.WriteLine("===================");
            Console.WriteLine(newnode1.OuterHtml);  //输出 <div></div>    注意 移除是从HtmlDocument中移除的，再次获取获取不到了

            HtmlNode newnode2 = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[2]/div[2]/p");
            Console.WriteLine(newnode2.OuterHtml);
            //<p class="no_tip">她还没有设置不能忍受清单　
            //    <a href="javascript:invite(5971070,8,'邀请设置不能忍受');" class="link_b needlogin">邀请她设置</a>
            //</p>
            newnode2.Remove();    //从文档树中移除newnode2节点
            HtmlNode newnode3 = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[2]/div[2]/p");   //再次获取该节点
            //Console.WriteLine(newnode3.OuterHtml);  //报未将对象引用到对象的实例异常，明显是找不到了，

            HtmlNode newnode4 = doc.DocumentNode.SelectSingleNode("/html/body/div[4]/div[1]/div[1]/div/div[1]/p[2]/b[1]");
            Console.WriteLine(newnode4.OuterHtml);
            //<b>相册：
            //    <a href="/photo/6971070.html" class="red">4</a>张
            //</b>
            HtmlNode node17 = HtmlNode.CreateNode("<div>再次创建一个节点</div>");
            newnode4.PrependChild(node17);      //跟AppengChild类似，只是插入位置不同PrependChildren接受一个节点集合，一次过插入多个节点而已
            Console.WriteLine(newnode4.OuterHtml);  
            //输出
            //<b>相册：
            //    <div>再次创建一个节点</div>
            //    <a href="/photo/6971070.html" class="red">4</a>张
            //</b>
            HtmlNode node16 = newnode4.SelectSingleNode("child::a[1]");
            HtmlNode node18 = HtmlNode.CreateNode("<p>新建一行</p>");
            newnode4.ReplaceChild(node18, node16);
            Console.WriteLine(newnode4.OuterHtml);
            //输出
            //<b>相册：
            //    <div>再次创建一个节点</div>
            //    <p>新建一行</p>张      //留意到node16代表得节点已经被替换掉了
            //</b>

            HtmlNode node19 = newnode4.SelectSingleNode("child::p[1]");
            node19.SetAttributeValue("class","class1");
            Console.WriteLine(node19.OuterHtml);    //输出 <p class="class1">新建一行</p>

            Console.WriteLine(HtmlNode.IsOverlappedClosingElement("<a>我爱你</a>"));   //输出 False
            Console.WriteLine(HtmlNode.IsCDataElement("<a>我爱你</a>"));   //输出 False
            Console.WriteLine(HtmlNode.IsClosedElement("<a>我爱你</a>"));   //输出 False
            Console.WriteLine(HtmlNode.IsEmptyElement("<a>我爱你</a>"));   //输出 False
            Console.WriteLine(newnode4.OuterHtml);


            HtmlNode node20 = HtmlNode.CreateNode("<p>新的第二行</p>");
            newnode4.AppendChild(node20);
            HtmlNodeCollection hnc = newnode4.SelectNodes("//p");   //根据XPath一次过获取多个Node
            Console.WriteLine(hnc.Count);   //输出29

            string str1 = node20.WriteContentTo();
            Console.WriteLine(str1);    //输出 新的第二行  将节点内容写入字符串

            //public void WriteContentTo(TextWriter outText);
            //public string WriteTo();
            //public void WriteTo(TextWriter outText);
            //public void WriteTo(XmlWriter writer);
            Console.ReadKey();
        }
{% endhighlight %}
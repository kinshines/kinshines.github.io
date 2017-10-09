---
layout: post
title: Elasticsearch搜索引擎搭建及应用
author: kinshines
date:   2017-05-07
categories: db
permalink: /archivers/elastic-search-apply
---

<p class="lead"><a href="https://www.elastic.co">Elasticsearch</a>是当今比较流行的搜索引擎，前段时间搭建了一套用于App资源的搜索服务，今天有时间整理一下，本文以Elasticsearch 5.4.0版本为例，参考官方文档<a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html">Elasticsearch Reference</a>
本文假设以搜索博客(blog)文章为需求基础，搜索时要求基于blog 的 title 和 content 两个维度作模糊匹配
</p>

### 基本概念
关于Elasticsearch，有几个核心的概念需要理解，理解这几个概念有助于加快学习进程
1. Near Realtime (NRT) 近乎实时
    Elasticsearch是一个近乎实时的搜索平台，这意味着从你索引了一个文档到能够搜索到它只有很短的延迟（通常1秒钟）
2. Cluster 集群
    集群是由一个或多个节点（服务器）来共同存储数据，并提供索引和搜索服务，一个集群通过集群名称来唯一标识，默认是elasticsearch，一个节点在加入一个集群时也需要通过集群名称来识别

    确保在同一环境中，不同集群切不可同名，否则节点可能会加入到错误的集群中去，例如分别使用logging-dev, logging-stage 和 logging-prod 作为dev,stage,prod这3个环境的集群名称
3. Node 节点
    节点是集群中一台独立的服务器，存储数据，参与索引和搜索服务。节点也是通过节点名称来唯一标识，通过配置集群名称来加入某个集群
4. Index 索引
    索引是有着相同的特征的文档的集合，比如，我们可以在同一个集群中创建用户数据的索引，产品目录的索引，以及订单数据的索引，索引通过索引名称（须英文小写）来唯一标识，索引的概念类似于我们关系型数据库中的"Database数据库"
5. Type 类型
    在一个索引内部，可以创建多个类型。类型用来定义具有相同字段的文档集合，例如在一个博客系统中，在一个索引中，为用户数据创建一个类型，为博客数据创建一个类型，为评论数据创建一个索引。类型的概念类似于我们关系型数据库中的"Table表"
6. Document 文档
    文档是是索引数据的最小单元，我们可以为一条用户数据创建一个文档，使用JSON结构表示
7. Shards & Replicas 分片和副本
    索引数据往往数据量达到单个节点的硬盘存储上限，如果将所有数据都存放在一个节点上，势必会影响搜索性能。
    Elasticsearch 提供分片的机制来解决这个问题，将索引数据分配到不同的分片上，通过分片实现数据的横向扩展，向不同分片分发搜索请求又可以提高性能

    副本机制通过节点间的副本保证了搜索服务的高可用性，搜索服务可以通过副本执行，这样就扩展了搜索吞吐量

    默认情况下，Elasticsearch的每一个索引分配了5个主分片和1个副本，这意味着至少需要两个节点，索引将拥有5个主分片和5个副分片（组成1个副本），共同形成了10个分片

### 下载和安装
Elasticsearch依赖Java 8，首先需要安装[jdk 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

安装完成后需要配置环境变量JAVA_HOME为本机jdk的安装目录，例如

        D:\Java\jdk1.8.0_112

并且在原先环境变量Path中末尾追加：

        ;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
        
[Elasticsearch下载地址](https://www.elastic.co/downloads/elasticsearch)，解压至D:\Java\elasticsearch-5.4.0 目录下，运行之前，先修改以下默认配置，打开 \config\elasticsearch.yml 文件，可以看到每一行都以#注释了，所以在修改了某一项后，需要把#去掉，使之生效

首先需要修改第17行，集群名称：

        cluster.name: my-application

第23行，节点名称：

        node.name: node-1

在部署到生产环境时，往往通过部署多个节点搭建集群，因此集群名称须一致，但各个节点需要分别命名，需要注意的是，如果dev,stage,prod环境是在同一网络里，需要将这三个环境的集群名称分别设置，决不能一样，否则会导致3个环境的数据混淆。

第55行，配置当前机器的内网IP地址：

        network.host: 192.168.0.1

第59行，ES服务端口号，默认就是9200，不建议修改

        http.port: 9200

第68行，集群节点列表，host1,host2分别配置为各个节点的IP：

        discovery.zen.ping.unicast.hosts: ["host1", "host2"]

ES对外提供服务使用的是9200端口，ES各节点内部通信使用的是9300端口，部署时需要告知运维开通9200和9300这两个端口

配置完毕，打开cmd窗口，输入命令：

        D:
        cd Java\elasticsearch-5.4.0\bin
        elasticsearch.bat

控制台显示一系列的加载信息，最后会显示如下，表示启动成功。

        [o.e.n.Node               ] [node-1] started

### 查看状态
#### 查看集群健康状态
在浏览器中输入：

        http://localhost:9200/_cat/health?v

返回：

epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent

1494167910 22:38:30  my-application green           1         1      0   0    0    0        0             0                  -                100.0%

#### 查看节点状态
在浏览器输入：
        
        http://localhost:9200/_cat/nodes?v

返回：

ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name

127.0.0.1            7          84   8                          mdi       *      node-1

#### 查看索引数据
在浏览器输入：
        
        http://localhost:9200/_cat/indices?v

返回：

health status index uuid pri rep docs.count docs.deleted store.size pri.store.size

没有数据，说明我们的集群目前还没有任何索引数据

### 创建Index

使用Postman发送PUT请求：

        http://localhost:9200/forum?pretty

再次发送GET请求：

        http://localhost:9200/_cat/indices?v

得到如下响应：

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size

yellow open   forum TfuBu8gvSFqVTQuFTqtUNg   5   1          0            0       650b           650b

### 配置IK分词插件
[elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)是一款优秀的中文分词插件，通过[ik-release](https://github.com/medcl/elasticsearch-analysis-ik/releases) 找到与elasticsearch相匹配的版本，如果你使用的ES版本太新，ik的作者尚未提供对应版本的发行版，则只能git checkout 源码后使用marven重新编译获得对应的插件

将下载或编译好的zip包 elasticsearch-analysis-ik-5.4.0.zip解压至elasticsearch-5.4.0\plugins\analysis-ik目录下

解压后在config目录下放置了ik分词所用到的词典，词典文件列表如下：
* extra_main.dic
* extra_single_word.dic
* extra_single_word_full.dic
* extra_single_word_low_freq.dic
* extra_stopword.dic
* main.dic
* preposition.dic
* quantifier.dic
* stopword.dic
* suffix.dic
* surname.dic

需要注意的是，以extra_为前缀的5个词典默认是不加载到词库的，如果想让ik自动加载，那么需要修改
config目录下的配置文件 IKAnalyzer.cfg.xml

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">extra_main.dic;extra_single_word.dic;extra_single_word_full.dic;extra_single_word_low_freq.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">extra_stopword.dic</entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
{% endhighlight %}

### 创建mapping
使用Postman发送POST请求：

        http://localhost:9200/forum/blog/_mapping

Body=>raw:
{% highlight js %}
{
    "blog": 
    {
        "_all": 
        {
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word",
            "term_vector": "no",
            "store": "false"
        },
        "properties": 
        {
            "title": 
            {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word",
                "include_in_all": "true"
            },
            "content": 
            {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word",
                "include_in_all": "true",
            }
        }
    }
}
{% endhighlight %}

### .Net 应用 ElasticSearch 搜索

首先，需要两个nuget package：

        Install-Package Elasticsearch.Net 
        Install-Package NEST

然后写一个与索引对应的Model 类：

{% highlight java %}
    [ElasticsearchType(IdProperty = "id", Name = "blog")]
    public class Blog
    {
        public int id { get; set; }
        public string title { get; set; }
        public string content { get; set; }
        public DateTime timestamp { get; set; }
    }
{% endhighlight %}

在初始化索引的过程中，考虑到数据量较大，一般采用bulk 方法操作List来初始化索引

#### Create

        client.Bulk(u => u.Index("forum").CreateMany<Blog>(list));

#### Update 

        var bulkQuest = new BulkRequest() { Operations = new List<IBulkOperation>() };
        foreach (var v in lst)
        {
                var operation = new BulkUpdateOperation<Blog, object>(v.id);
                operation.Doc = v;
                operation.Index = "forum";
                bulkQuest.Operations.Add(operation);
        }
        client.Bulk(bulkQuest);

#### Delete

        client.Bulk(u => u.Index("forum").DeleteMany<Blog>(list));

#### Search

        var request = new SearchRequest("forum", "blog")
            {
                From = from-1,
                Size = to-from+1,
                Query = new DisMaxQuery()
                {
                    Queries = new List<QueryContainer>()
                    {
                        new MatchQuery() {Field = "title", Query = keywords},
                        new MatchQuery() {Field = "content", Query = keywords}
                    }
                }
            };
            var response = client.Search<Blog>(request);
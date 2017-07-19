---
layout: post
title: Elasticsearch修改请求队列容量
author: kinshines
date:   2017-07-19
categories: db
permalink: /archivers/elasticsearch-queue-capacity
---

<p class="lead">今天ES搜索一直报 Too many request 的错误，详细的错误信息：</p>

        "type": "es_rejected_execution_exception",
        "reason": "rejected execution of org.elasticsearch.transport.TransportService$4@1f9512f on EsThreadPoolExecutor[search, queue capacity = 1000, org.elasticsearch.common.util.concurrent.EsThreadPoolExecutor@c0efba[Running, pool size = 7, active threads = 7, queued tasks = 1000, completed tasks = 573995]]"

从错误信息中推断，ES默认的请求队列容量是1000，类似于IIS应用程序池里的队列长度，超过这个容量后的请求就得不到响应了，因此需要修改ES的queue capacity，以增加产能

使用POSTMAN发送HTTP PUT请求，URL:  

        http://localhost:9200/_cluster/settings

Request Body=>raw内容：

        {
            "persistent":{
                "threadpool.search.queue_size":8000
            },
            "transient":{
                "threadpool.search.queue_size":8000
            }
        }

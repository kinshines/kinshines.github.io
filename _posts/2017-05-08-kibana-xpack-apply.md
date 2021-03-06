---
layout: post
title: Elasticsearch 搭建数据可视化Kibana及监控插件x-pack
author: kinshines
date:   2017-05-08
categories: db
permalink: /archivers/kibana-xpack-apply
---

<p class="lead">上一节记录了Elasticsearch的搭建，这一节继续整理Elasticsearch数据可视化Kibana及监控插件x-pack的搭建</p>

### 下载和安装Kibana

[Kibana下载地址](https://www.elastic.co/downloads/kibana)，解压至D:\Java\kibana-5.4.0-windows-x86 目录下，运行之前，先修改以下默认配置，打开 \config\kibana.yml 文件：

{% highlight yml %}
# Kibana is served by a back end server. This setting specifies the port to use.
#server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
#server.host: "localhost"

# Enables you to specify a path to mount Kibana at if you are running behind a proxy. This only affects
# the URLs generated by Kibana, your proxy is expected to remove the basePath value before forwarding requests
# to Kibana. This setting cannot end in a slash.
#server.basePath: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayloadBytes: 1048576

# The Kibana server's name.  This is used for display purposes.
#server.name: "your-hostname"

# The URL of the Elasticsearch instance to use for all your queries.
#elasticsearch.url: "http://localhost:9200"

# When this setting's value is true Kibana uses the hostname specified in the server.host
# setting. When the value of this setting is false, Kibana uses the hostname of the host
# that connects to this Kibana instance.
#elasticsearch.preserveHost: true

# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"

# The default application to load.
#kibana.defaultAppId: "discover"

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
#elasticsearch.username: "user"
#elasticsearch.password: "pass"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# Optional settings that provide the paths to the PEM-format SSL certificate and key files.
# These files validate that your Elasticsearch backend uses the same key files.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
#elasticsearch.ssl.verificationMode: full

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 0

# Time in milliseconds to wait for Elasticsearch at Kibana startup before retrying.
#elasticsearch.startupTimeout: 5000

# Specifies the path where Kibana creates the process ID file.
#pid.file: /var/run/kibana.pid

# Enables you specify a file where Kibana stores log output.
#logging.dest: stdout

# Set the value of this setting to true to suppress all logging output.
#logging.silent: false

# Set the value of this setting to true to suppress all logging output other than error messages.
#logging.quiet: false

# Set the value of this setting to true to log all events, including system usage information
# and all requests.
#logging.verbose: false

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000.
#ops.interval: 5000

# The default locale. This locale can be used in certain circumstances to substitute any missing
# translations.
#i18n.defaultLocale: "en"

{% endhighlight %}

可以看到每一行都以#注释了，所以在修改了某一项后，需要把#去掉，使之生效

首先看第2行，Kibana监听的端口号：5601，这里不建议修改

        server.port: 5601

第7行，服务器地址，改为本机的IP地址，如：192.168.0.1

        server.host: "localhost"

第18行，节点名称：

        server.name: "your-hostname"
        
第21行，要连接的elasticsearch 地址，如：192.168.0.1:9200

        elasticsearch.url: "http://localhost:9200"

配置完毕，运行 bin/kibana.bat

打开浏览器，访问：http://localhost:5601

### 安装x-pack

1.	进入Elasticsearch 安装目录，安装 x-pack

            bin/elasticsearch-plugin install x-pack

2.	启动ElasticSearch

            bin/elasticsearch

3.	进入Kibana安装目录，安装x-pack

            bin/kibana-plugin install x-pack

4.	启动Kibana

            bin/kibana

5.	浏览器访问Kibana

          http://localhost:5601/

    默认用户名：elastic
    默认密码：changeme

6.	申请x-pack License
    初始安装的x-pack为试用许可证，试用期为30天，需要更新为Basic Licensen，申请地址：
    [Register for a Free Basic License](https://register.elastic.co/)

    申请成功后，Elastic会将许可证发送至申请邮箱，许可证实际上是一个json文件，有效期1年

7. 更新 License
    使用Postman 发送PUT请求URL：
    
            http://localhost:9200/_xpack/license?acknowledge=true

    Body=>raw 中填入上一步得到的json内容

    Authorization=>Type选择Basic Auth
    
    Username填写：elastic
    Password填写：changeme

    Send后正确响应：

            {
                "acknowledged": true,
                "license_status": "valid"
            }

8. 查看 License
    使用Postman发送GET请求，URL：
            
            http://localhost:9200/_xpack/license

    Send后可以查看当前许可证的有效日期








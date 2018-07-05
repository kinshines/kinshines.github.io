---
layout: post
title: MySQL 8.0 Windows zip 安装过程
author: kinshines
date:   2018-07-05
categories: db
permalink: /archivers/mysql-8-zip-install
---

<p class="lead">MySQL 8.0 已正式发布，据官方提供的性能测试，8比5快两倍，于是迫不及待地尝试，之前写过一篇基于MySQL 5.6.36 的文章，相比于8.0在安装方式上有了较大的变化，遂附加一遍文章更新，本文以当前最新版8.0.11为例，在Windows 10 环境上演示</p>

### 下载解压
[MySQL Community Server 8.0.11](https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.11-winx64.zip)
推荐下载免安装的ZIP Archive，解压到自定义的目录下

### 配置文件
解压后发现相比于5.6.36版本，安装目录缺少默认的ini文件，需要我们自行添加 my.ini，内容如下：

{% highlight xml %}

[mysqld]
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
 
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
 
# These are commonly set, remove the # and set as required.
basedir = D:\Program Files\mysql-8.0.11-winx64
datadir = D:\Program Files\mysql-8.0.11-winx64\data
port = 3306
# server_id = .....
 
 
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M 
 
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
 
character-set-server = utf8mb4
 
performance_schema_max_table_instances = 600
table_definition_cache = 400
table_open_cache = 256
[mysql]
default-character-set = utf8mb4
[client]
default-character-set = utf8mb4


{% endhighlight %}

注意，里面的 basedir 是我本地的安装目录，datadir 是我数据库数据文件要存放的位置，各项配置需要根据自己的环境进行配置。

查看所有的配置项，可参考：[参考mysqld-option-tables](https://dev.mysql.com/doc/refman/8.0/en/mysqld-option-tables.html)

### 初始化数据库
在MySQL安装目录的 bin 目录下执行命令：

        mysqld --initialize --console

执行完成后，会打印 root 用户的初始默认密码，比如：

        2018-04-20T02:35:01.507037Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
        2018-04-20T02:35:01.507640Z 0 [System] [MY-013169] [Server] D:\Program Files\mysql-8.0.11-winx64\bin\mysqld.exe (mysqld 8.0.11) initializing of server in progress as process 11064
        2018-04-20T02:35:01.508173Z 0 [ERROR] [MY-010340] [Server] Error message file 'D:\Program Files\mysql-8.0.11-winx64\data\share\english\errmsg.sys' had only 1090 error messages, but it should contain at least 4512 error messages. Check that the above file is the right version for this program!
        2018-04-20T02:35:05.464644Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: APWCY5ws&hjQ
        2018-04-20T02:35:07.017280Z 0 [System] [MY-013170] [Server] D:\Program Files\mysql-8.0.11-winx64\bin\mysqld.exe (mysqld 8.0.11) initializing of server has completed

其中，第4行的“APWCY5ws&hjQ”就是初始密码，在没有更改密码前，需要记住这个密码，后续登录需要用到，要是你手贱，关快了，或者没记住，那也没事，删掉初始化的 datadir 目录，再执行一遍初始化命令，又会重新生成的。

[参考data-directory-initialization-mysqld](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization-mysqld.html)

### 安装服务
在MySQL安装目录的 bin 目录下执行命令：

        mysqld --install [服务名]

后面的服务名可以不写，默认的名字为 mysql。当然，如果你的电脑上需要安装多个MySQL服务，就可以用不同的名字区分了，比如 mysql5 和 mysql8。

安装完成之后，就可以通过命令net start mysql启动MySQL的服务了。

[参考windows-start-service](https://dev.mysql.com/doc/refman/8.0/en/windows-start-service.html)

### 更改密码和密码认证插件
在MySQL安装目录的 bin 目录下执行命令：

        mysql -uroot -p

这时候会提示输入密码，记住了第3步的密码，填入即可登录成功，进入MySQL命令模式。

在MySQL8.0.4以前，执行

        SET PASSWORD=PASSWORD('[修改的密码]');

就可以更改密码，但是MySQL8.0.4开始，这样默认是不行的。因为之前，MySQL的密码认证插件是“mysql_native_password”，而现在使用的是“caching_sha2_password”。

因为当前有很多数据库工具和链接包都不支持“caching_sha2_password”，为了方便，我暂时还是改回了“mysql_native_password”认证插件。

在MySQL中执行命令：

        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

修改密码验证插件，同时修改密码。

如果想默认使用“mysql_native_password”插件认证，可以在配置文件中配置default_authentication_plugin项。

        [mysqld]
        default_authentication_plugin=mysql_native_password

[参考upgrading-from-previous-series](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password)

后续的添加环境变量以及其他的SQL操作与之前版本相同，就不再赘述，链接[MySQL 使用汇总](https://kinshines.github.io/archivers/mysql-summary)
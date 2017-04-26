---
layout: post
title: MySQL 使用汇总
author: kinshines
date:   2016-10-02
categories: db
permalink: /archivers/mysql-summary
---

<p class="lead"> <a href="https://www.mysql.com/">MySQL</a> 是当今最流行的开源数据库，尽管.NET领域使用SQL SERVER 居多，但在很多情况下也免不了会使用MySQL，本文是我在使用MySQL 过程中的学习汇总，以5.6.36版本为例</p>

### 下载解压
[MySQL Community Server](https://dev.mysql.com/downloads/mysql/)
推荐下载免安装的ZIP Archive，解压到自定义的目录下，修改my-default.ini文件：
* basedir = D:\Program Files\mysql-5.6.36-winx64
* datadir = D:\Program Files\mysql-5.6.36-winx64\data
* port = 3306
* character_set_server=utf8mb4

### 添加环境变量
1. 右键单击我的电脑->属性->高级系统设置(高级)->环境变量

    点击系统变量下的新建按钮

    输入变量名：MYSQL_HOME

    输入变量值：D:\Program Files\mysql-5.6.36-winx64

 2. 选择系统变量中的Path

     点击编辑按钮，在原变量值末尾增加  ;%MYSQL_HOME%\bin

     注意是在原有变量值后面加上这个变量，用;隔开，不能删除原来的变量值

### 注册windows服务并启动
1. 从控制台进入到MySQL解压目录下的 bin 目录下
2. 输入服务安装命令：

        mysqld install MySQL --defaults-file="D:\Program Files\MySQL\mysql-5.6.36-winx64\my-default.ini"

    安装成功后会提示服务安装成功

    移除服务命令为：mysqld remove
3. 启动服务命令为：net start mysql

    提示MySQL服务已经启动成功

    同理，停止服务命令为：net stop mysql

### 修改root账号的密码
刚安装完成时root账号默认密码为空，此时可以将密码修改为指定的密码。如：123456

    c:\>mysql –uroot

    mysql>show databases;

    mysql>use mysql;

    mysql>UPDATE user SET password=PASSWORD("123456") WHERE user='root';

    mysql>FLUSH PRIVILEGES;

    mysql>QUIT

此时回到command窗口

 输入命令：mysql -uroot -p 输入密码后再次进入mysql命令窗口

如果MySQL连接端口不是3306，甚至连接服务器不是本机，可以使用下面的命令：

mysql -h机器名或IP地址 -P端口号 -u root -p

### 开启MySQL远程连接
 确定服务器上的防火墙没有阻止 3306 端口

 允许远程连接 MySQL 用户并授权

    mysql>grant all PRIVILEGES on discuz.* to ted@'123.123.123.123' identified by '123456';

上面的语句表示将 discuz 数据库的所有权限授权给 ted 这个用户，允许 ted 用户在 123.123.123.123 这个 IP 进行远程登陆，并设置 ted 用户的密码为 123456 。

下面逐一分析所有的参数：

all PRIVILEGES 表示赋予所有的权限给指定用户，这里也可以替换为赋予某一具体的权限，例如：select,insert,update,delete,create,drop 等，具体权限间用“,”半角逗号分隔。

discuz.* 表示上面的权限是针对于哪个表的，discuz 指的是数据库，后面的 * 表示对于所有的表，由此可以推理出：对于全部数据库的全部表授权为“*.*”，对于某一数据库的全部表授权为“数据库名.*”，对于某一数据库的某一表授 权为“数据库名.表名”。

ted 表示你要给哪个用户授权，这个用户可以是存在的用户，也可以是不存在的用户。

123.123.123.123 表示允许远程连接的 IP 地址，如果想不限制链接的 IP 则设置为“%”即可。

123456 为用户的密码。

执行了上面的语句后，再执行下面的语句，方可立即生效。

    mysql>flush privileges;

同理，想让myuser使用mypassword从任何主机连接到mysql服务器的话：

    mysql>GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword';

    mysql>flush privileges;

如果想允许用户myuser从ip为192.168.1.3的主机连接到mysql服务器，并使用mypassword作为密码:

    mysql>GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword’;

    mysql>flush privileges;

还有一种不常用的改表法，这里也一并介绍一下：

    c:\>mysql -u root -pvmware

    mysql> use mysql;

    mysql> update user set host = ‘%’ where user = ‘root’;

    mysql> select host, user from user;

    mysql> flush privileges;

### MySQL的备份与还原
MySQL备份和还原,都是利用mysqldump、mysql和source命令来完成的
1. 备份
    在cmd窗口，使用命令“mysqldump  -u 用户名 -p databasename >exportfilename”导出数据库到文件，例如mysqldump -u root -p voice>voice.sql，然后输入密码即可开始导出
2. 还原
    输入命令：mysql -uroot -p 输入密码后再次进入mysq命令窗口
    输入命令"show databases；"查看当前有哪些数据库
    建立新数据库，输入"create database voice；"
    切换到刚建立的数据库，输入"use voice；"
    导入数据，输入"source voice.sql；"
    开始导入，再次出现"mysql>"并且没有提示错误即还原成功



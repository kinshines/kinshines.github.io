---
layout: post
title: npm常用配置
author: kinshines
date:   2018-09-14
categories: cmd
permalink: /archivers/npm-config
---

<p class="lead">最近在学习Vue,需要常用到npm,本文是常用配置的总结
</p>

### 命令行方式
#### 修改global路径和cache路径
首先，我们可以配置npm的全局模块的存放路径以及cache路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立"node_global"及"node_cache"两个文件夹。

启动cmd，输入

    npm config set prefix "C:\Program Files\nodejs\node_global"
    npm config set cache "C:\Program Files\nodejs\node_cache"

#### 修改源地址
由于npm主服务器在国外，国内用户下载安装包时会失败，建议将源地址修改为taobao的

    npm config set registry https://registry.npm.taobao.org

#### 查看更改结果

    npm config ls -l
    npm config list

### 配置文件方式
在用户主目录下添加文件.npmrc

添加文件内容

registry=https://registry.npm.taobao.org/
prefix=C:\Program Files\nodejs\node_global
cache=C:\Program Files\nodejs\node_cache
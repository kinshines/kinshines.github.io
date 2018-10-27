---
layout: post
title: 使用ReadtheDocs、Sphinx搭建托管文档环境
author: kinshines
date:   2018-10-23
categories: dev
permalink: /archivers/read-the-doc-sphinx
---

<p class="lead">程序员最讨厌的两件事：一是讨厌写文档，而是讨厌别人的程序不写文档。大约是因为写文章是一件枯燥而乏味的事，下面将介绍如何优雅地写文档和给别人看文档。使用 Sphinx + GitHub + ReadtheDocs 作为文档写作工具，用 Sphinx 生成文档，GitHub 托管文档，再导入到 ReadtheDocs</p>

### Sphinx
Sphinx是一个基于Python的文档生成项目，最早只是用来生成 Python 官方文档，随着工具的完善， 越来越多的知名的项目也用他来生成文档，甚至完全可以用他来写书采用了reStructuredText作为文档写作语言, 不过也可以通过模块支持其他格式，待会我会介绍怎样支持MarkDown格式。

### 安装Sphinx

        pip install sphinx sphinx-autobuild sphinx_rtd_theme

### 初始化
        
        mkdir cookbook
        cd  cookbook
        sphinx-quickstart

下面是我填写的，其他基本上默认即可：

        Separate source and build directories (y/n) [n]:y 
        Project name: cookbook 
        Author name(s): Xiong Neng 
        Project release [1.0]: 0.2.2 
        Project language [en]: zh_CN

添加一篇文章，在source目录下新建hello.rst，内容如下:

        hello,world
        =============

index.rst 修改如下:

        Contents:
        .. toctree::
        :maxdepth: 2

        hello

#### 更改主题 sphinx_rtd_theme
更改source/conf.py:

        import sphinx_rtd_theme
        html_theme = "sphinx_rtd_theme"
        html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]

toctree 支持多级目录,比如要想将python.rst,swift.rst笔记在不同的目录,toctree这样设置:

        Contents:

        .. toctree::

        python/python
        swift/swift





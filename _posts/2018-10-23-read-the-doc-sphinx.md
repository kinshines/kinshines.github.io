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

### 更改主题 sphinx_rtd_theme
更改source/conf.py:

        import sphinx_rtd_theme
        html_theme = "sphinx_rtd_theme"
        html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]

toctree 支持多级目录,比如要想将python.rst,swift.rst笔记在不同的目录,toctree这样设置:

        Contents:

        .. toctree::

        python/python
        swift/swift

### 预览效果
然后在更目录执行make html，进入build/html目录后用浏览器打开index.html

### 支持markdown编写
通过recommonmark 来支持markdown

        pip install recommonmark

然后更改conf.py:

        from recommonmark.parser import CommonMarkParser
        source_parsers = {
            '.md': CommonMarkParser,
        }
        source_suffix = ['.rst', '.md']

然后修改刚刚的hello.rst，改用熟悉的hello.md编写:

        ## hello world
        ### test markdown

再次运行make html后看效果，跟前面一样。

### GitHub托管
一般的做法是将文档托管到版本控制系统比如github上面，push源码后自动构建发布到readthedoc上面， 这样既有版本控制好处，又能自动发布到readthedoc，实在是太方便了。

具体几个步骤非常简单，参考官方文档：https://github.com/rtfd/readthedocs.org:

1. 在Read the Docs上面注册一个账号
2. 登陆后点击 “Import”.
3. 给该文档项目填写一个名字比如 “scrapy-cookbook”, 并添加你在GitHub上面的工程HTTPS链接, 选择仓库类型为Git
4. 其他项目根据自己的需要填写后点击 “Create”，创建完后会自动去激活Webhooks，不用再去GitHub设置
5. 一切搞定，从此只要你往这个仓库push代码，readthedoc上面的文档就会自动更新.

注：在创建read the docs项目时候，语言选择”Simplified Chinese”

在构建过程中出现任何问题，都可以登录readthedoc找到项目中的”构建”页查看构建历史，点击任何一条查看详细日志

### FAQ
build的时候出现错误：! Package inputenc Error: Unicode char 我 (U+6211)

解决办法，在conf.py中添加:

{% highlight txt %}

latex_elements={# The paper size ('letterpaper' or 'a4paper').
'papersize':'a4paper',# The font size ('10pt', '11pt' or '12pt').
'pointsize':'12pt','classoptions':',oneside','babel':'',#必須
'inputenc':'',#必須
'utf8extra':'',#必須
# Additional stuff for the LaTeX preamble.
'preamble': r"""
\usepackage{xeCJK}
\usepackage{indentfirst}
\setlength{\parindent}{2em}
\setCJKmainfont{WenQuanYi Micro Hei}
\setCJKmonofont[Scale=0.9]{WenQuanYi Micro Hei Mono}
\setCJKfamilyfont{song}{WenQuanYi Micro Hei}
\setCJKfamilyfont{sf}{WenQuanYi Micro Hei}
\XeTeXlinebreaklocale "zh"
\XeTeXlinebreakskip = 0pt plus 1pt
"""}

{% endhighlight %}

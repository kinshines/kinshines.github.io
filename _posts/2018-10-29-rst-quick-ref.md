---
layout: post
title: reStructuredText(rst)快速入门语法说明
author: kinshines
date:   2018-10-29
categories: dev
permalink: /archivers/rst-quick-ref
---

<p class="lead">reStructuredText 是扩展名为.rst的纯文本文件，含义为"重新构建的文本"，也被简称为：RST或reST；是Python编程语言的Docutils项目的一部分，Python Doc-SIG (Documentation Special Interest Group)。</p>
<p class="lead">.rst 文件是轻量级标记语言的一种，被设计为容易阅读和编写的纯文本，并且可以借助Docutils这样的程序进行文档处理，也可以转换为HTML或PDF等多种格式，或由Sphinx-Doc这样的程序转换为LaTex、man等更多格式。
</p>

# 行内样式
## 斜体
重点、解释文字

        *重点(emphasis)通常显示为斜体*
        `解释文字(interpreted text)通常显示为斜体`

*重点(emphasis)通常显示为斜体*
## 粗体
重点强调

        **重点强调(strong emphasis)通常显示为粗体**

**重点强调(strong emphasis)通常显示为粗体**

## 等宽
        
        ``行内文本(inline literal)通常显示为等宽文本，空格可以保留，但是换行不可以。``

行内文本(inline literal)通常显示为等宽文本，空格可以保留，但是换行不可以。

## 章节标题
章节头部由下线(也可有上线)和包含标点的标题 组合创建, 其中下线要至少等于标准文本的长度。


可以表示标题的符号有 =、-、`、:、'、"、~、^、_ 、* 、+、 #、<、> 。


对于相同的符号，有上标是一级标题，没有上标是二级标题。


标题最多分六级，可以自由组合使用。


全加上上标或者是全不加上标，使用不同的 6 个符号的标题依次排列，则会依次生成的标题为H1-H6。

        =========
        一级标题
        =========
        二级标题
        =========

        一级标题
        ^^^^^^^^
        二级标题
        ---------
        三级标题
        >>>>>>>>>
        四级标题
        :::::::::
        五级标题
        '''''''''
        六级标题
        """"""""

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

---
layout: post
title: Git报错 remote HTTP Basic Access denied的解决方法
author: kinshines
date:   2021-03-16
categories: develop
permalink: /archivers/git-access-denied
---

<p class="lead">Git 执行 git pull 命令是报错如下：</p>

        remote: HTTP Basic: Access denied
        fatal: Authentication failed for 'http://localhost/rep.git/'

问题原因：账号密码验证不通过，密码或者权限不对，导致 Git 操作失败。

解决方案：输入：

        git config --system --unset credential.helper

再次进行 Git 操作，输入正确的用户名，密码即可。
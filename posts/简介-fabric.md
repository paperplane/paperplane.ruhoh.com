---
title: 'Fabric系列之简介'
date: '2012-10-08'
description:
categories: ['Fabric','OPS']
---

##简介##

Fabric 既是一个 python(2.5+)类库也是一个基于 SSH 的应用程序部署和系统管理任务的命令行工具。

它提供整套本地和远程执行 shell 命令、上传下载文件操作，以及一些辅助功能比如提示和执行用户输入，放弃执行之类。

更具体点就是：

+ Fabric 是一个能让你通过命令行工具执行任意 python 函数的工具。

+ Fabric 是一个能使你通过 ssh 执行 shell 命令更简单化和 pythonic 的子例程库。

通常，我们会结合这两项，使用 fabric 来写和执行 python 函数，或者说是任务(task)，来自动和远程服务器交互。典型的用法就是创建一个包含一个或多个函数的 python 模块，然后通过命令行工具 fab 来执它们。

一个简单但完整的fabric例子是这样的：首先定义python模块fabfile/fabfile.py：

    from fabric.api import run    

    def host_type():    
            run('uname -s')

然后命令行执行 fab 以及结果：

    $ fab -H localhost host_type
    [localhost] run: uname -s
    [localhost] out: Linux
    Done.
    Disconnecting from localhost... done.

这是一个单主机单任务的执行例子。

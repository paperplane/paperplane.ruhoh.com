---
title: Fab Command Line
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##工具命令--Fab Command Line##

使用fabric最通用的方式是利用它的命令行工具--fab,安装fabric时这个需要添加到shell可执行路径下（一般默认pip install或easy_install都会）。fab同其他unix citizen一样，使用标准风格命令行切换，帮助输出等等。

在大多数情况下，fab可以直接调用而没有options,跟随一个或多个参数，这些是任务名(task name)。正如我们在前面所看到的，fab会在当前路径寻找fabfile,然后执行其中的task1和task2。

    $ fab task1 task2

fab有很多命令行参数和选项供我们选择，而且很灵活。

####执行任意远程shell命令####

fabric使用一种很少熟知的命令行规则来执行shell命令，正如下面这种模式：

    $ fab [options] -- [shell command]

若我们需要在example.com上执行uname -a 这个命令只需（此时并不需要fabfile中定义任何任务）：

    $ fab -H example.com -- uname -a

它等价于以下fabfile:

    from fabric.api import run
    env.hosts = [‘example.com’]
    def anonymous():
        run("uname -a")

***

####单个任务命令行参数####

以下是一些使用规则：

    用冒号区分任务名和任务参数
    用逗号隔开不同任务参数
    用等号为每个任务参数赋值
    用分号隔开不同主机名

    def new_user(username, admin='no', comment="No comment provided"): 
         log_action("New User (%s): %s" % (username, comment))    
         pass

以下都是正确调用：

    $ fab new_user:myusername
    $ fab new_user:username=myusername
    $ fab new_user:myusername,yes
    $ fab new_user:myusername,admin=yes
    $ fab new_user:myusername,admin=no,comment='Gary\, new developer (starts Monday)'
     
通过上面的方式我们给任务传进了它需要的参数，按照执行模式所讲，但我们还没有给该任务执行执行相应的主机列表。(host, hosts, role and roles)这些关键字与任务本身无关，但是去要用来指定任务需要执行的主机列表。这时候正确的指定主机方式如：

    $ fab new_user:myusername,hosts="host1;host2"

当然可以使用roles,或者-H之类，需要注意语法。逗号（、），分号（；），引号(””)的使用。

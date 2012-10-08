---
title: 'Fabric系列之环境字典'
date: '2012-10-08'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##环境字典##

Fabric 的一个简单但是完整部分就是"environment"，我的理解就是类似于平时所说的环境变量：这是一个 python 字典的子类，用来作为组合设置的注册表或者是共享任务内部数据的命名空间。

环境字典当前版本(1.4.3)是作为一个全局单例(global singleton)来实现,即fabric.state.env，被包括在 fabric.api 中。key-value 形式，其中的 keys 又称为 ENV变量(以后不加说明都称 ENV 变量)。一般用法有以下几种。

###ENV AS Configure###

大多数fabric的行为可以通过改变ENV变量来控制,我们初始化Fabric相关的设置也就是初始化这些 ENV 变量。比如 ENV.HOSTS。其他常见的包括：
    User：fabric 建立 ssh 连接时默认使用本地用户名，但是可以使用 env.user来覆盖该默认设置。还可以对 host 列表中每个 host 分别指定。
    Password：在需要的时候用来显示指定默认连接或者 sudo 的密码，如果未设置，fabric 会在必要的时候提示输入。
    Warn_only：这个设置是一个 boolean 值，用来决定 fabric 在远程主机执行任务时检测到错误时的行为：是放弃执行还是仅仅提示。

####作用域####

这种设置既包括全局也包括局部的，很多时候我们只想暂时给某一个局部块改变其 ENV 变量值。这个时候就可以通过上下文管理器(Context Manager)中的settings，这个只会在它所包括的块中改变 ENV 变量的值。全局使用很简单，直接在定义函数之前设置或者通过 Fab 命令行初始化全局 ENV 变量值。比如在所有 函 数 定 义 之 前 指 定 ： 
    env.hosts = [‘root@ns1.buaa.edu.cn’,‘root@ns2.buaa.edu.cn’]。
局部作用域例子：
    from fabric.api import settings, run
    
    def exists(path):
            with settings(warn_only=True, host_string=’host1’):
                    return run('test -e %s' % path)

###ENV as Shared State###
                    
Env 对象是一个简单的字典子类，因此在 Fabfile 代码中也可能存储相息。这样在很多时候当我们执行通过一次执行执行多个任务时就能共享一些任务间信息。典型的使用就是共享主机的信息。
    from fabric.api import run,env

    def set_hosts():
            env.hosts = ['host1',host2]        
    
    def test_task():
            run('uname -a')
                              
    $fab set_hosts test_task

###Other Consideration###

前面对 ENV 变量值的读写基于属性访问方式，但是 ENV 也是 python 字典的子类，ENV 变量的值还有字典模式的访问方式。

    print("Executing on %s as %s" % (env.host, env.user))
    print("Executing on %(host)s as %(user)s" % env)

####环境字典的历史####

过去 fabric 还不是纯 python 的时候，ENV 变量不同任务（task）间交流共享信息的唯一途径。现在你可以直接调用其他任务或者子例程，甚至保存模块级共享状态。将来，fabric 也许会线程安全，这时 ENV 变量也许是唯一容易/安全的保存全局变量途径。

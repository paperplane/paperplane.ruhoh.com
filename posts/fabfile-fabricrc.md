---
title: Fabfile & Fabricrc
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##两个文件--Fabfile & Fabricrc##

####Fabfile####

之前已经说过很多次fabfile, 也提到过这是一个python模块，但是也不去对。这需要知道fabric执行原理。fabric能够加载python模块（如fabfile.py）或者包（比如fabfile/目录下包含__init__.py）。默认情况下，它会寻找名叫fabfile或fabfile.py的东西。

Fabfile的搜索算法是需找用户当前活动目录，或者它的父目录。因此一般在一个项目中，把fabfile放在该项目源代码的根路径则在程序中任何地方都能使用fab命令。

Fabric现在是纯python,所以可以随时可以以任何方式import它的组件，为了封装与方便，现在所有7类核心API都被包括在模块fabril.api下。所有只需在fabfile开头中：

    from fabric.api import *

当然普通API(Contrib API)需要另外导入。在写fabfile时，本着实用和简便的原则，选择是使用from fabric.api import * 还是 from fabric.api import abort, cd, env, get 具体情况具体分析。


***

####Fabricrc####

我们之前一直说fabfile, 如果有人不想用fabfile这个名字怎么办，因为他想用一个他自己喜欢的名字，或者处于其他目的。这个时候同样有办法，那就是在fabric本地配置文件fabricrc中修改，添加如下条目：

    fabfile=anything you want

默认情况下fab命令都会遵循一定原则去找fabfile或fabfile.py，但若通过fabricrc文件里面增加：fabfile=task.py，fab命令就会直接去寻找task.py，而不是fabfile了。

Fabricrc是fabric简单的用户配置文件，它会包含一些key-value形式的内容，一条信息一行，每行类似于：user = ssh_user_name这种形式。

一般不需要该配置文件，默认路径在env.fabfile中指定有：~/.fabricrc，这个配置会被fab -c命令选项覆盖。对于一些固定不变的env变量通过fabricrc设置可能比较方便。需要注意的是fabricrc中指定的fabfile文件名会直接影响到fab命令查找fabfile的文件名，正因为它的优先级在fabfile之上，所以该文件中key-value的形式将直接改变env中的值。

使用举例：

如果你的SSH LOGIN 用户名不同于你的工作间用户名，而你又不想在项目的fabfile中改变env.user的值（也许你想方便其他用户），这是你可以使用如下设置：

    user = ssh_user_name

这个当你运行fab时，你的fabfile会将env.user设置为’ssh_user_name’

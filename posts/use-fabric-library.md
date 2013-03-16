---
title: Library Use
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##使用类库--Library Use##

Fabric最基本的用法是通过fabfiles和fab工具，而且这几乎占据整个官方文档内容。正如在开始简介中所介绍，fabric自身就是一个python类库，在文档中有一小部分也介绍了当fabric作为类库的使用。这部分就像官方文档所介绍一样，只是一个草稿，它主要讨论了fabric建立连接和断开连接的原理，对如何作为类库来使用，以及使用举例几乎没有谈到。更多的是建议直接阅读fabric源码中的main.py从使用fab来找到使用方式。

那么在python程序中如何使用fabric API呢.

***

####Fab + Fabfile + OS####

最容易想到的方式就是使用fabfile和fab工具，然后结合python的OS模块执行命令，用popen或subprocess来执行fab命令，但这显得很笨拙和混乱。

***

####For + With + Env.host_string####

然后想到就是完全抛弃fab命令和python的OS模块，如果不按照fabfile和fab命令行工具这种方式使用，直接使用fabric的API，最大的问题就是如何初始化fab中的env变量,因为env.hosts等变量需要fab来初始化。在前面曾经提到过一种做法就是使用上下文管理器with,在其中设置host_string，这时多个hosts就可以通过循环来实现，但是这明显违背了我们使用fabric的初衷。

***

####Local + Fab + Set_hosts + …####

因为主要是初始化env.hosts这样的变量，后来决定专门写一个任务来设置ENV变量，这样将待执行主机作为这个任务的参数传递进去。因为本地任务不需要显式指定初始化主机列表，所以设定主机列表的任务作为一个本地任务来执行，即local(‘fab set_hosts’)，其他任务可以完全作为fab的第二个，第三个参数，…这样实现多个主机执行多个任务也成为可能。

***

####Execute + Tasks + Hosts####

上面这种方式，如果加以延伸其实就很灵活，我们可以把需要的变化的东西都作为参数，然后通过local来执行fab命令。这样避免了循环给host string赋值。后来发现还有更直接的方式就是完全抛弃fab命令行工具，使用fabric中有个叫execute的API，这个函数第一个参数为任务名，后面我们可以通过host/hosts/role/roles作为参数指定主机。这种结合前一种定义任务传参的方式，应该是最好的方式。即直接调用execute(‘taskname’,hosts=[‘a’,’b’])。

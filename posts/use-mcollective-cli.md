---
title: Use Mcollective CLI
date: '2013-03-15'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---
##使用MCollective命令行应用程序##

命令行接口是MCollective最早也是最重要的设计。大多数情况下都是和这个称为MCO的可执行命令交互，它有很多子命令，参数和标志值。

***

####MCO命令基本用法####

安装配置完MCollective之后，可以用一个简单的命令来做测试：mco ping
    
    [root@master ~]# mco ping
    master.example.com                       time=50.93 ms
    web.example.com                          time=91.21 ms
    puppet.example.com                       time=92.17 ms

    ---- ping statistics ----
    3 replies max: 92.17 min: 50.93 avg: 78.10

在上面的例子中ping的子命令对应的是一个应用程序。这些应用程序对应的就是在默认路径/usr/libexec/mcollective/mcollective/application中的ruby程序。如果需要查看所有命令列表，你可以通过命令mco help。下面显示的列表中除了默认安装的命令还有已经安装的代理。
    
    [root@master ~]# mco help
    The Marionette Collective version 2.0.0
      controller      Control the mcollective daemon
      facts           Reports on usage for a specific fact
      find            Find hosts matching criteria
      help            Application list and help
      inventory       General reporting tool for nodes, collectives and subcollectives
      package         Install and uninstall software packages
      ping            Ping all nodes
      plugin          MCollective Plugin Application

对于某个命令（也就是应用程序）具体的用法可以使用mco help application这样的格式 或者mco application --help来查看帮助说明。下面示例rpc应用程序。
    
    [root@master ~]# mco help rpc
    Generic RPC agent client application
    Usage: mco rpc [options] [filters] --agent <agent> --action <action> [--argument <key=val> --argument ...]
    Usage: mco rpc [options] [filters] <agent> <action> [<key=val> <key=val> ...]
            --no-results, --nr           Do not process results, just send request
        -a, --agent AGENT                Agent to call
    ......

上面的帮助说明首先展示了基本的命令行语法，接着是对每个选项的具体说明。通过这些你可以看到在大都数应用程序中通用的选项和主机过滤器的使用方法。
***
####执行RPC请求####

#####请求概述#####

应用程序rpc是一个主要的应用程序用于向所有的server发出请求。它能够与很多标准远程过程调用(RPC)代理交互。下面是一个在多台server机器上显示httpd服务start的例子。
    
    [root@master ~]# mco rpc service start service=httpd
    Determining the amount of hosts matching filter for 2 seconds .... 3
     * [ ============================================================> ] 3 / 3
    puppet.example.com                       Request Aborted
       Could not start Service[httpd]: Execution of '/sbin/service httpd start' returned 1:
    web.example.com                          Request Aborted
       Could not start Service[httpd]: Execution of '/sbin/service httpd start' returned 1:

    Finished processing 3 / 3 hosts in 1736.54 ms


在这个过程中事件发生的顺序如下：

    执行网络发现并找到多台servers

    发送请求并显示回复的进度条

    显示任何出现的结果，比如执行未成功之类

    显示执行的统计信息

MCollective客户端应用程序旨在提供最相关最重要的信息。在这种情况下，如果应用程序执行过程中出现个别失败的情况下，结果显示中就会说明在哪些机器上执行失败，而不会返回正确执行的信息。这很多情况下也取决于你的程序如何返回。下面的介绍默认你已经安装那些WIKI中提到的常用的代理。

#####请求详细#####

MCollective代理被分解为一个个动作(action)和每个动作需要传入的参数(arguments)有时候有些动作也不需要参数。

    % mco rpc service start service=httpd

上面显示了一个RPC命令的基本组成：

使用应用程序rpc。这是一个通用的应用程序，可以和任何代理交互。

请求会被发送到广播域中所有拥有service代理的机器。

向service代理发送stop动作的请求

在stop动作上应用参数值httpd。

这条命令与下面这条完全等同：
    
    % mco rpc --agent service --action start --argument service=httpd

#####发现可用代理#####

上面的命令展示了service代理的交互，对于你安装的所有代理可以通过MCollective系统自带的一个应用程序plugin来获取整个列表
    
    [root@master ~]# mco plugin doc
    Please specify an agent. Available agents are:
      helloworld      Echo service for MCollective
      package         Install and uninstall software packages
      process         Agent To Manage Processes
      rpcutil         General helpful actions that expose stats and internals to SimpleRPC clients
      service         Start and stop system services

列表显示计算机发现的所有代理，为了显示完全（即有第二列的内容），每个代理必须安装本地DDL文件。
显示代理的所有动作（actions），输入（inputs）,输出（outputs）可以再次使用应用程序plugin：
    
    [root@master ~]# mco plugin doc service
    service
    =======
    Start and stop system services

          Author: R.I.Pienaar
         Version: 2.1
         License: ASL2
         Timeout: 60
       Home Page: https://github.com/puppetlabs/mcollective-plugins

    ACTIONS:
    ========
       restart, start, status, stop

       restart action:
       ---------------
           Restart a service

           INPUT:
               service:
                  Description: The service to restart
                       Prompt: Service Name
                         Type: string
                   Validation: ^[a-zA-Z\.\-_\d]+$
                       Length: 90


           OUTPUT:
               status:
                  Description: The status of the service after restarting
                   Display As: Service Status

这显示了service自动生成的帮助文档的一部分。首先是目标数据（metadata）部分，比如version,author和license之类信息。接着是整个可用动作（actions）列表。更详细就是关于每个动作，需要的输入参数和输出结果。比如status需要输入一个service,这是一个String,长度不能超过30。它的输出返回每台机器的服务状态。

不同于之前说的，这里每个成功执行的结果都返回了，因为这个特定的动作是在检索信息，所以MCollective假设想知道全部的所有的数据，而不论成功还是失败。

***
####使用过滤器选择请目标####

#####基本过滤器#####

MCollective一个关键性能就是快速发现网络资源。发现规则就是使用过滤器filters来写。
    
    % mco rpc service status service=httpd -S "environment=development or customer=acme"

这指明了过滤器规则为：限制RPC请求目标执行主机或者是Puppet环境为development或者是属于Customer acme。

filter可以基于facts，节点的配置管理类别（Configuration Management Class），节点的标识（identity）或者是安装的代理。

下面是一些简单的例子和相应的描述。


![图片]({{urls.media}}/mcollective/mco-cli.png)


正如所看到的，filter可以使用agent，class和/或fact。可以在几乎所有地方使用正则表达式。可以在一个命令里组合filter使用这样所有的条件都必须匹配。
    
    [root@master ~]# mco ping -W "/default/ country=~^uk"
    web.example.com                          time=58.80 ms

    ---- ping statistics ----
    1 replies max: 58.80 min: 58.80 avg: 58.80

这里/default/匹配class, country=~^uk匹配facts

#####复杂组合或者select查询#####

前面的例子都很简单，而且受限于组合使用。如果想使用更复杂的布尔逻辑就可以使用-S选项创建搜索。
    
    [root@master ~]# mco ping -S "default and not country=acme"
    master.example.com                       time=50.93 ms
    web.example.com                          time=91.21 ms
    puppet.example.com                       time=92.17 ms

    ---- ping statistics ----
    3 replies max: 92.17 min: 50.93 avg: 78.10

-S布尔操作符允许使用and/or/not(!)来建立搜索逻辑。这是组合之前哪些简单选项办不到的。

#####使用数据插件（data plugin）来过滤#####

在2.1版本中，自定义的数据插件也能被用来创建复杂的过滤器filter:（现在还不是很成熟）
    
    % mco ping -S "fstat('/etc/hosts').md5=/baa37/ and environment=development"

这个例子会搜索指定文件md5哈希值和development环境同时匹配。可以通过mco plugin doc查询可用数据插件。

对于数据插件查看需要输入和提供输出时使用mco plugin doc fstat命令。目前，每个数据函数在匹配时可以仅仅接受一个输入而且限制每次调用输出单一字段。

***

####链接RPC请求####

RPC程序能够一个接一个链接。下面的例子展示使用package代理来发现具有特定版本号的mcollective的机器然后调度这些机器运行Puppet。
    
    % mco rpc package status package=mcollective -j| 
    jgrep "data.properties.ensure=2.0.0-6.el6" |mco rpc puppetd runonce

Mcollective的结果能够通过使用开源工具gem,jgrep来过滤。MCollective输出与jgrep完全兼容。

#####查看原始数据#####

RPC应用程序默认尽量显示读友好的数据。如果相查看真正原始输出数据，可以通过-v标识来停用显示帮助（diable display helper）。
    
    [root@master mcollective]# mco rpc package status package=mcollective -v
    Determining the amount of hosts matching filter for 2 seconds .... 3
     * [ ============================================================> ] 3 / 3


    puppet.example.com                      : OK
        {:provider=>"yum",     :release=>"1.el5",     :output=>"",     :version=>"2.0.0",     :arch=>"noarch",     :epoch=>"0",     :name=>"mcollective",     :ensure=>"2.0.0-1.el5"}

    web.example.com                         : OK
        {:provider=>"yum",     :release=>"1.el5",     :output=>"",     :version=>"2.0.0",     :arch=>"noarch",     :epoch=>"0",     :name=>"mcollective",     :ensure=>"2.0.0-1.el5"}

    master.example.com                      : OK
        {:provider=>"yum",     :release=>"1.el6",     :output=>"",     :version=>"2.0.0",     :arch=>"noarch",     :epoch=>"0",     :name=>"mcollective",     :ensure=>"2.0.0-1.el6"}

    ---- package#status call stats ----
               Nodes: 3 / 3
         Pass / Fail: 3 / 0
          Start Time: Tue Jul 24 16:09:06 -0400 2012
      Discovery Time: 2002.99ms
          Agent Time: 195.06ms
          Total Time: 2198.05ms

这个数据同样可以以JSON格式返回。
    
    [root@master mcollective]# mco rpc package status package=mcollective -j
    [
      {
        "action": "status",
        "agent": "package",
        "data": {
          "provider": "yum",
          "release": "1.el6",
          "output": "",
          "version": "2.0.0",
          "arch": "noarch",
          "epoch": "0",
          "name": "mcollective",
          "ensure": "2.0.0-1.el6"
        },
        "statuscode": 0,
        "statusmsg": "OK",
        "sender": "master.example.com"
      },
      {
        "action": "status",
        "agent": "package",
        "data": {
          "provider": "yum",
          "release": "1.el5",
          "output": "",
          "version": "2.0.0",
          "arch": "noarch",
          "epoch": "0",
          "name": "mcollective",
          "ensure": "2.0.0-1.el5"
        },
        "statuscode": 0,
        "statusmsg": "OK",
        "sender": "puppet.example.com"
      },
      {
        "action": "status",
        "agent": "package",
        "data": {
          "provider": "yum",
          "release": "1.el5",
          "output": "",
          "version": "2.0.0",
          "arch": "noarch",
          "epoch": "0",
          "name": "mcollective",
          "ensure": "2.0.0-1.el5"
        },
        "statuscode": 0,
        "statusmsg": "OK",
        "sender": "web.example.com"
      }
    ]


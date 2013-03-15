---
title: Using Filter
date: '2013-03-15'
description:
categories:
---
##使用过滤器##

使用过滤器来控制发现和寻址是MCollective中关键概念之一。可以使用facts,classes,agents和server identity作为过滤器或者它们的组合来减少能够影响的主机范围。

可以通过mco rpc --help来查看客户端支持的所有过滤器。

    Host Filters
        -W, --with FILTER                Combined classes and facts filter
        -S, --select FILTER              Compound filter combining facts and classes
        -F, --wf, --with-fact fact=val   Match hosts with a certain fact
        -C, --wc, --with-class CLASS     Match hosts with a certain config management class
        -A, --wa, --with-agent AGENT     Match hosts with a certain agent
        -I, --wi, --with-identity IDENT  Match hosts with a certain configured identity
    
注意两点：

所有的过滤器支持正则表达式而且一些支持比较操作符大于小于不等于这些。

如果组合使用过滤器，它们之间是and（‘且’）的关系，意味着所有节点必须满足所有条件。

***

####Fact过滤器####

Facts过滤器需要安装fact plugin。下面示例用法：

这是检索在facts中type=>Notebook的server, 然后安装zsh。后两个例子是使用正则表达式过滤以Note开头的type值得server。

    [root@master mcollective]# mco rpc package install package=zsh -F type=Notebook
    Determining the amount of hosts matching filter for 2 seconds .... 1
     * [ ============================================================> ] 1 / 1

    Finished processing 1 / 1 hosts in 72312.84 ms

    [root@master mcollective]# mco rpc package install package=zsh -F type=/Note/
    Determining the amount of hosts matching filter for 2 seconds .... 1
     * [ ============================================================> ] 1 / 1

    Finished processing 1 / 1 hosts in 147.10 ms

    [root@master mcollective]# mco rpc package install package=zsh -F type=~^Note
    Determining the amount of hosts matching filter for 2 seconds .... 1
     * [ ============================================================> ] 1 / 1

    Finished processing 1 / 1 hosts in 183.57 ms

####Agent,Identity和Class过滤器####

    [root@master mcollective]# mco ping -I /example/
    master.example.com                       time=49.60 ms
    puppet.example.com                       time=88.94 ms
    web.example.com                          time=89.48 ms

    ---- ping statistics ----
    3 replies max: 89.48 min: 49.60 avg: 76.01

    [root@master mcollective]# mco ping -C /de/
    puppet.example.com                       time=61.51 ms
    web.example.com                          time=101.05 ms
    master.example.com                       time=102.04 ms

    ---- ping statistics ----
    3 replies max: 102.04 min: 61.51 avg: 88.20

    [root@master mcollective]# mco ping -A helloworld
    master.example.com                       time=55.17 ms
    puppet.example.com                       time=94.54 ms

    ---- ping statistics ----
    2 replies max: 94.54 min: 55.17 avg: 74.86
    
####组合fact和class过滤器####

    [root@master mcollective]# mco ping -W "/default/ type=Notebook"
    master.example.com                       time=49.76 ms

    ---- ping statistics ----
    1 replies max: 49.76 min: 49.76 avg: 49.76


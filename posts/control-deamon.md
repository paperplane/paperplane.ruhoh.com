---
title: Control Daemon
date: '2013-03-15'
description: ['Mcollective','OPS']
categories: ['Mcollective','Daemon']
---

##控制守护进程##

在每个节点运行的守护进程保持着内部状态并且支持重新加载所有的代理，这些功能由工具mco controller提供。它负责在客户端与任意运行的守护进程交互。

如果想重新加载所有代理而不重启守护进程，可以向该进程发送USR1信号。

也可以发送USR2信号来循环日志级别的DEBUG到TATAL，仅仅不停发送信号然后查看日志。

####统计####

这可以显示守护进程的基本统计数据

    [root@master mcollective]# mco controller stats
    Determining the amount of hosts matching filter for 2 seconds .... 3
                          master.example.com> total=30, replies=29, valid=30, invalid=0, filtered=0, passed=30
                          puppet.example.com> total=8, replies=7, valid=8, invalid=0, filtered=0, passed=8
                             web.example.com> total=8, replies=7, valid=8, invalid=0, filtered=0, passed=8
    Finished processing 3 / 3 hosts in 66.16 ms
     
每个字段的含义分别如下：

    统计    含义
    total   从中间件接收的消息总数
    replies 返回的回复
    valid   消息通过有效性检查
    invalid 消息未通过验证
    filtered    消息接收但因为过滤器而未回复数目

####重新加载特定代理####


这种情况时想加载某个代理而不重启整个守护进程，经常用于新添加某个代理。

    [root@master mcollective]# mco controller reload_agent --arg rpcutil
    Determining the amount of hosts matching filter for 2 seconds .... 3

                             web.example.com> reloaded rpcutil agent
                          puppet.example.com> reloaded rpcutil agent
                          master.example.com> reloaded rpcutil agent

    Finished processing 3 / 3 hosts in 77.21 ms

####重新加载所有代理####

像之前一样，针对所有代理重新加载。这与向进程发送USR1信号一样

    [root@master mcollective]# mco controller reload_agents
    Determining the amount of hosts matching filter for 2 seconds .... 3

                             web.example.com> reloaded all agents
                          puppet.example.com> reloaded all agents
                          master.example.com> reloaded all agents

    Finished processing 3 / 3 hosts in 181.20 ms


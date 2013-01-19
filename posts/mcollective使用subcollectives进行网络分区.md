---
title: Mcollective系列之使用SubCollectives进行网络分区
date: '2013-01-16'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---
<strong>[Mcollective中文文档目录](http://paperplane.ruhoh.com/documentation/mcollective/)</strong>

分区既是为了解决单一广播域问题，也是对安全的一种考虑。

***

###概述###

默认情况下，所有的servers属于单一广播域，如果你有一个代理安装在你的网络中所有的机器上，那么你发送给有这个代理的机器的消息就会直接被所有机器收到而无论使用滤器(filters)，这在常规使用中能工作的很好但是在下面这些场景中会出现问题：

+ A.拥有一个非常大而拥挤的网络。假设有成千上万机器需要每隔10秒回复请求，大量的消息会被创建，你想给流量分区。

+ B.不同的地方拥有多个数据中心，网站流量和延迟很大。你想在每个数据中心复制监控和自动化管理，但是所有都在一个广播域仍会看到大量流量缓慢链路。

+ C.需要完成多个独立安装，但仍想保持MCollective中心管理功能。

+ D.平面网络(plat network)安全问题。SimpleRPC拥有一个安全框架提醒用户和网络拓扑但是核心网络没有。

Sub collectives允许你定义广播域并且配置一个mcollective属于这些域中的一个或多个。

* * *

###分区策略###

确定如何分区比较复杂，需要考虑的有：对消息流的理解，请求者所在位置和中间件集群拓扑。很重要的一点需要注意就是大多数中间件能够仅仅在它们"兴趣"的区域发送流量。因此如果你在1000台机器的10台中安装一个代理，那么只有这10台机器能够收到相关流量。

下面是一个52个节点collective的例子，collective在4个国家很多数据中心都有机器。有3个ActiveMQ连接成网状。另有一个国家在图中未画出。这些ACtiveMQ节点同时也是Puppet Master,Nagios实例或者其他共享设施组件。

![图片]({{urls.media}}/mcollective/mcollective-sub1.png)

这个网络理想的安装可能是：

<strong>在所有节点安装MCollective NRPE和Puppeted Agent

3个ActiveMQ地点是Puppet Master

Nagios在每个区域监测该区域机器<

区域间流量相互隔离

系统管理员和注册数据保持覆盖所有collectives的能力</strong>

现在的问题是这3个Puppet master都会有一份所有流量的拷贝，即使这不是它们请求的。它们之间的连接会有大量流量经过，这在小数量节点可以处理，但是节点多了就无法处理。必须创建单独广播域。

使用SubCollectives可以实现：

<strong>一个全局的collective包含所有的机器

UK，DE，ZA和US collectives仅仅包含它们所在区域的机器

EU collectives包含UK，DE和ZA所有机器</strong>

![图片]({{urls.media}}/mcollective/mcollective-sub2.png)

然后在配置Nagios和Puppet Master仅仅与这些 sub mcollectives通信，这些collectives的流量只会在它们内部。
下面显示US ACtiveMQ实例在分区前和分区后流量变化图：

![图片]({{urls.media}}/mcollective/mcollective-sub3.png)

* * *
###配置MCollective###

配置分区collective很简单，上面的DE节点(该节点的Server.cfg)可以配置如下：

    collectives = mcollective,de_collective,eu_collective
    main_collective = mcollective

collectives 表示节点所属的所有collectives，而main_collective 表示应该去哪注册消息

<strong>使用测试</strong>

    [root@master ~]# mco inventory --list-collectives
    Collective                     Nodes
    ==========                     =====
    web                            1
    local                          1
    puppet                         2
    mcollective                    3
                     Total nodes: 3
    
    [root@master ~]# mco ping -T mcollective
    master.example.com                       time=52.75 ms
    web.example.com                          time=92.25 ms
    puppet.example.com                       time=92.83 ms
    ---- ping statistics ----
    3 replies max: 92.83 min: 52.75 avg: 79.28

***
###分区为了安全###

另一个使用分区的优势就是安全。虽然在SimpleRPC框架中有一个安全模型能够注意拓扑结构但核心网络层没有。即使你仅仅给某人权限来执行SimpleRPC请求，他依然可以使用mco ping来发现网络中其他节点。

通过创建submcollectives你可以有效而且完备地在现有网络上创建隔离区，并在中间件层给予限制。

<strong>ActiveMQ过滤</strong>

之前的配置在很多情况下足够，但是若想有效阻止Sub Collective流量在网络间传播，需要重新对ACtiveMQ配置。通过定义网络连接和添加过滤器来实现。

    <networkConnectors>
        <networkConnector
            <excludedDestinations>
                <topic physicalName="us_collective.>" />
                <topic physicalName="uk_collective.>" />
                <topic physicalName="de_collective.>" />
                <topic physicalName="za_collective.>" />
                <topic physicalName="eu_collective.>" />
            </excludedDestinations>
            name="us-uk"
            uri="static:(tcp://stomp1.uk.my.net:6166)"
            userName="amq"
            password="secret"
            duplex="true" 
        />
    </networkConnectors>

这种配置就会使 US<->UK的连接不再传输us_collective 的流量。

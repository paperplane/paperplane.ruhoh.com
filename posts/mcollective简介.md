---
title: 'Mcollective系列之简介'
date: '2012-11-08'
description: 
categories: ['Mcollective','OPS']
tags: 'Mcollective'
---

##简介## 

MCollective 是一个构建服务器编排(Server Orchestration)和并行工作执行系统的框架。

首先，MCollective 是一种针对服务器集群进行可编程控制的系统管理解决方案。在这一点上，它的功能类似：Func，Fabric  和 Capistrano。

其次，MCollective 的设计打破基于中心存储式系统和像 SSH 这样的工具，不再仅仅痴、迷于 SSH 的 For 循环。它使用发布订阅中间件(Publish Subscribe Middleware)这样的现代化
工具和通过目标数据(meta data)而不是主机名(hostnames)来实时发现网络资源这样的现代化理念。提供了一个可扩展的而且迅速的并行执行环境。

有文档指出： MCollective 工具为命令行界面，但它可与数千个应用实例进行通信，而且传输速度惊人。无论部署的实例位于什么位置，通信都能以线速进行传输，使用的是一个类似多路传送的推送信息系统。MCollective 工具没有可视化用户界面，用户只能通过检索来获取需要应用的实例。Puppet Dashboard 提供有这部分功能。

***

* MCollective特点

1 能够与小到大型服务器集群交互

2 使用广播范式(broadcast paradigm)来进行请求分发，所有服务器会同时收到请求，而只有与请求所附带的过滤器匹配的服务器才会去执行这些请求。没有中心数据库来进行同步，网络是唯一的真理

3 打破了以往用主机名作为身份验证手段的复杂命名规则。使用每台机器自身提供的丰富的目标数据来定位它们。目标数据来自于：Puppet, Chef, Facter, Ohai 或者自身提供的插件

4 使用命令行调用远程代理

5 能够写自定义的设备报告

6 大量的代理来管理包，服务和其他来自于社区的通用组件

7 允许写 SimpleRPC 风格的代理、客户端和使用 Ruby 实现 Web UIs

8 外部可插件化(pluggable)实现本地需求

9 中间件系统已有丰富的身份验证和授权模型，利用这些作为控制的第一道防线。

10 重用中间件来做集群、路由和网络隔离以实现安全和可扩展安装。

***

* 插件化内核 

MCollective 就是一个框架，一个空壳。它除了 MCO 命令之外都可以被替换被自定义。MCollective 提供了一个可扩展稳定的面向需求的核心框架，可以根据需要在以下几方面构建自己的系统：

1 替换 Stomp 兼容中间件，选择其他基于 AMQP 的中间件

2 替换授权系统，使用符合自身需求的系统

3 替换基于 Ruby 和 YAML 的序列化，可以选择想 JSONSchema 这样的中间语言。

4 增加数据源，现在支持 Chef 和 Puppet，可以提供插件以访问自己的工具数据(tools data)。

5 社区已经有了关于 Facter 和 Ohai 的插件。

6 利用 MCollective 作为传输定期运行和分发存储数据(inventory data)来创建中心存储(Central Inventory)服务。

7 现在社区提供的插件以代理为主，还有关于 Fact Source,审计，授权，注册，安全，工具类。

---
title: Mcollective系列之常用术语
date: '2013-01-16'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---

这一部分主要包括了后面经常会用到的MCollective术语。

最重要的概念是server,client。这与C/S架构中客户端/服务器端没有必然关系，在MCollective中是多个server<----->一个client的结构

+ Server

服务端，简单说就是数据提供方。它是mcollective守护进程，一个负责托管代理和管理中间件连接的应用程序服务器。
+ Client

客户端，简单说是数据/信息接收方。它是创建命令让代理去执行的应用程序，特别地这会是一台安装有客户端程序的机器用户可以使用像mco ping这样的命令来与代理交互。通常客户端会使用MCollective::Client库来与Collective通信。

+ Node

节点，server运行的计算机或操作系统

+ Agent

代理，Ruby代码块，执行一个特定的角色，mcollective主要用来托管代理。代理可以执行防火墙操作，服务管理，包管理等任务。它在所有的‘server’节点上执行命令，将返回传递给‘client’。MCollective提供内建的过滤机制来选择执行代理。

+ Action

动作，代理执行的任务称为动作。每个代理就像exim队列管理代理可能执行mailq, rm, retry等任务一样。这些动作由代理提供。

+ Plugins

插件，ruby代码块，位于server机器并且执行像安全，连接处理，代理等的特定角色。按照维基百科解释就是一系列增强/提高其他软件应用程序的软件并且通常不能独立运行。

+ Middleware

中间件，这里仅指类似Apache ActiveMQ这样的发布/订阅中间件 

处理服务间通信的软件，这个服务传输请求消息和回复消息。可以看成是一个有着路由规则的邮件服务器。

+ Message Quence

消息队列，可以从中读取或向里写入的数据存储应用程序。就像SMTP服务器中的邮件队列。

+ Connector

连接器，MCollective::Connector类型的插件，处理与所选中间件的连接

+ Name Space

命名空间，当前发送到中间件的消息直接在名为/topic/mcollective.package.command的主题下（topics）而回复则在/topic/mcollective.package.reply

这个例子中命名空间是”mcollective”,所有的servers和clients如果想形成同一个collective的一部分就必须使用相同命名空间。

中间件通常能够携带不同命名空间因此有多个collectives。

+ Collective

在同一个命名空间下操作的servers,nodes和中间件的组合

多个colletives能够通过使用不同命名空间共享同一中间件

+ Subcollective

一个server可以属于多个命名空间。一个subcollective就是一个命名空间，这个subcollective属于这个server所属全部collectives的子集

subcollectives用于在高流量网络中网络分区和控制广播域

+ Facts

节点信息，比如域名，国家，角色，操作系统版本等等

facts由MCollective::Facts类型的插件提供

+ Registration

注册，server向代理发送的信息称为注册。发送注册消息的代码是MCollective::Registration类型的插件。

+ Security

安全，MCollective::Security类型的插件来负责加密、验证和消息在传给connector时的编码。

+ User

用户，servers和clients都向中间件验证，用户一般指的是用于中间件验证的用户名

+ Aduilt

审计，与SimpleRPC相关，审计动作指将请求落到磁盘日志或者相关动作

+ Authorization

授权，与SimpleRPC相关，授权过程就是根据请求者的验证信息对请求进行允许或取消操作

+ MCollective

通过队列发送和接收消息的软件，代理在系统上执行的动作依赖消息的内容。就像是你的邮件客户端和你或者是人在聊天室里。

几个相关概念韦恩图：

![图片]({{urls.media}}/mcollective/mcollective-concept.png)

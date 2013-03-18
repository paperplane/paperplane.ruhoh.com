---
title: Mcollective系列之解决问题及使用原因
date: '2013-01-16'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---

这部分集中概括MCollective所解决问题及选择MCollective的原因，并比较MCollective与SSH循环的优势所在。并简单讨论MCollective给Puppet带来的改变。

+ 解决问题

系统发现(System discovery)

信息收集(Inventory Collection)

任务执行(Task Execution)

配置管理(Configuration Management)

包管理(Package Management)

+ 选择原因

去中心化存储(No Centralized inventory)

成千上万节点(Thousands Of Nodes)

新建框架(Framework For Creation)

异步执行(Asynchronous)

+ SSH循环比较

异步/事件驱动

可扩展性

使用facts,classes等，不仅仅是hostname/ip

模块化（安全、中间件，代理）

自动发现(Auto discovery)

去中心化存储(De-centralized inventory)

---
title: Mcollective系列之文档综述
date: '2013-01-16'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---
<strong>[Mcollective中文文档目录](http://paperplane.ruhoh.com/documentation/mcollective/)</strong>

文档共分为概述篇，使用篇，插件篇，原理篇，集成篇，编程篇 六大部分，各部分相互联系，相互引用。没有绝对的划分。本文档不讨论任何关于安装配置问题以及插件安装问题。划分时最头疼的是关于安全这部分的划分，原定计划增加安全篇部分。但考虑到具体安全涉及到中间件安全、安全插件，SimpleRPC安全（授权、认证和审计）等在相应插件，中间件和编程部分也会涉及。故将安全概述放在原理部分讨论，以期对整个流程有个完整认识，具体安全环节在具体环节再做讨论。

概述篇是对整个MCollective的总结性介绍，包括基本功能，特点，优点的总结性比较和一个完整的术语表。

使用篇是对MCollective用户接口MCO命令行工具进行完整介绍，包括原生支持的命令像ping/plugin/rpc/controller/mc-call-agent等以及安装常用插件命令的使用。这部分还重点讨论了过滤机制。

插件篇是对所有插件类型，以及编写该类型自定义插件的完整介绍。

集成篇会重点讨论MCollective推荐使用的中间件ACtiveMQ的相关内容，包括安全，集群等内容。这部分还将关注MCollective与Puppet等其他工具的集成。

编程篇会完整介绍SimpleRPC风格代理和客户端的编程，以及相关的验证、授权和审计问题。


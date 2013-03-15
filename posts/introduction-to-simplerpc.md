---
title: Introduction To SimpleRPC
date: '2013-03-16'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','SimpleRPC']
---
##SimpleRPC简介##

MCollective是一个编写所有代理（agents）和客户端(clients)功能的框架，它提供了一个丰富的系统来实现这些。

虽然MCollective原生的客户很低级，就像TCP/IP之于HTTP，TCP/IP原生客户端也没有提供任何验证、授权等等。

MCollective Simple RPC是一个在标准客户端之上的框架，抽象了复杂性并提供了许多规则和标准。这有点像HTTP在TCP/IP之上创建REST服务。

SimpleRPC框架提供如下（分别针对客户端及代理）：

    提供编写代理(agents)和客户端(clients)的简单规则，有利于自定义设计。
    很容易编写代理包括输入验证和错误时的合理反馈机制。
    提供调用代理的审计日志记录能力
    提供调用代理及动作的细粒度授权能力
    拥有数据描述语言（DDL）用于描述代理和自动生成用户接口时给出提示
    提供通用调用工具能够和大多数代理兼容
    仍可以自定义客户端，简单易行
    提供合理输出，DDL提供标准one-size-fits-all函数
    标准客户端所有性能
    核心格式（core format）之上的标准消息格式


---
title: Mcollective系列之消息流与消息格式
date: '2013-01-16'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---
<strong>[Mcollective中文文档目录](http://paperplane.ruhoh.com/documentation/mcollective/)</strong>

###消息流###

+ 消息流程

下图展示了MCollective系统的基本消息流。MCollective通过广播范式进行请求分配。客户端发送消息然后广播给整个广播域的节点。

![图片]({{urls.media}}/mcollective/mcollective-message.png)

整个流程如下：

步骤及描述

    中间件收到由管理员工作站发送的请求消息。请求消息附带有过滤器指明符合具有内容为cluster=c的fact的机器应该执行请求消息中的操作
    中间件网络将该消息广播给所有节点。中间件网络可以是有位于多个地点、网络和数据中心的多台服务器
    每个节点收到该消息然后验证过滤器
    只有符合条件cluster=c的机器才会执行请求消息的操作并发送回复消息，根据中间件最后只有开始发送请求消息的工作站会收到回复消息

* * *

###消息格式###

中间件获取的消息试图包括所有mcollective作用需要的信息，并且也避免了中间件中可能的特殊功能。这使为其他中间件创建连接插件(connector plugins)变得更容易。

目前消息编码和解码的功能由MCollective::Security::*中的类实现。将编码从安全插件中抽象出来的一个目标就是重构，这样在实现安全插件的时候只需要符合下面结构就可以。

通常情况下，这些对于开发者是隐藏的，特别是使用SimpleRPC，如果想实现自己的安全或序列化插件就需要准确知道这些是怎样结合在一起。

<strong>发送给代理的请求</strong>

![图片]({{urls.media}}/mcollective/mcollective-request.png)

+ :filter

除非通过消息将节点信息发送给代理，否则过滤器会被所有节点用于过滤。所有类型的过滤器定义在MCollection::Optionparser类中。

每个过滤器是个数组，而且你可已有多个过滤器，每种类型之间将是and的关系。

    CF Class
    
    这将会应用配置管理系统中classes/recipes/cookbooks/roles内容并匹配。

    filter["cf_class"] = ["common::linux"]

    Agent

    这将会匹配已有Agents列表

    filter["agents"]=["packages"]

    Facts

    因为facts都是key-value的形式，与前两种稍不同。这里需要创建嵌套的哈希

    filter["fact"]=[{:fact=>"country",:value=>"/uk/"}]

    1.1.0版本后支持更多的比较操作符：==，=～，<=,>=,><和！=

    filter["fact"] = [{:fact => "country", :value => "uk", :operator => "=="}]

    但是需要注意如果使用正则表达式匹配比较操作符同之前版本==一样，因为混合使用会有兼容性问题。

    Identity

    identity是守护进程server端配置文件中的配置选项，此选项是过滤server节点。许多主机可以有相同的identity,这仅仅是另一种过滤器而不是像hostname一样唯一标识主机。

    filter["idenity"]=["foo.bar.com"]
+ :senderid

配置文件中配置选项identity的值，这应该是客户端配置文件的配置选项。表示发送请求的客户端。

+ :msgtarget

发送给中间件的消息的topic或者channel。这个在1.3.1版本之后不再包括在消息中了。

+ :body

body的内容随着选择来实现的安全提供者的不同而不同，PSK安全提供者将body编码为序列化的形式等待传输。

这确保了端到端传输的多样化类型。其他安全提供者可能使用JSON,最后的解码依然由安全提供者实现所以这完全由提供者决定。

就SimpleRPC而言，整个RPC请求和回复都会被放在消息的body部分。

+ :hash

这仅用于PSK提供者来，来指定安全提供者，所以是可选的。

+ :msgtime

消息发送时间，unix时间戳

+ :ttl

每个请求都会有TTL，消息超过这个时间就会被放弃

+ :requestid

每个发送的消息的唯一标识，回复消息会附加同样的ID用于验证

<strong>从代理的回复</strong>

回复同请求很类似，只有很少不同。

![图片]({{urls.media}}/mcollective/mcollective-response.png)

一旦改回复消息被创建，安全插件就会序列化该回复然后将其发送给connector，如果是使用PSK安全插件这通常会使用Marshal或Yaml这样的工具来序列化，当然这取决于你的安全插件。

+ :senderagent

发送回复的代理的名字

+ :requestid

这是请求中的需要回复的ID。代理一般不生成消息仅仅回复，所以它总是被提供。
 

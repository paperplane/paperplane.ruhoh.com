---
title: ActiveMQ Middleware
date: '2013-03-16'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','ActiveMQ']

---

##ActiveMQ中间件##

此部分公分三小部分，即ActiveMQ安全，ActiveMQ TLS，ActiveMQ 集群。

***

####ActiveMQ 安全####

    
作为MCollective重要部分就是考虑安全。目前看到的例子都是允许所有的代理和所有的节点代理进行通信。问题是通过这种方法如果在某个节点有一个不受信任的用户他就能够安装一个客户端应用程序来从server配置文件中读取用户名/密码从而控制整个体系结构。

默认的消息主题格式与ActiveMQ Wildcard 模式相兼容，因此可以做细粒度的控制。

#####ActiveMQ Wildcard模式#####

ActiveMQ支持目的地通配符来提供简单的支持联合的名称层次结构。这个概念在金融市场流行很长时间，用来作为一种以层次结构组织事件（如价格变化）的方式并且用也通配符能够很容易订阅你所感兴趣的信息范围。

假设想通过证券交易所订阅发布价格消息，可能会使用如下结构：

    PRICE.STOCK.NASDAQ.ORCL 发布ORACLE在NASDAQ的股票价格
    PRICE.STOCK.NYSE.IBM   发布IBM在NYK证券交易所的股票价格

订阅者同样会使用相同目的地来订阅正需要的价格，或者使用通配符来定义层次模式匹配目的地。

<strong>通配符支持</strong>

使用如下通配符：

    .用来作为路径分隔符
    *用来匹配路径任意名称
    >用来递归匹配任意以这个名称开始的目的地

例如：

    订阅    含义
    PRICE.> 任意交易所任意产品的任意价格
    PRICE.STOCK.>   任意交易所一个股票的任意价格
    PRICE.STOCK.NASDAQ.*    NASDAQ的任意股票价格
    PRICE.STOCK.*.IBM   任意交易所的任意IBM股票价格

<strong>自定义路径分隔符</strong>

允许自定义之后，可以使用FOO/BAR/*来代替FOO.BAR.*

仅需要如下设置：
    <plugins>
       .....
       <destinationPathSeparatorPlugin/>
    </plugins>

***
 
#####MCollective中ActiveMQ安全设置#####

在MCollective中默认的消息目的地看起来如下：

    /topic/mcollective.agentname.command
    /topic/mcollective.agentname.reply

如果使用subcollectives，每个subcollective都会有如下形式主题（topics）：

    /topic/subcollective.agentname.command
    /topic/subcollective.agentname.reply

对于某个节点属于某个属于sub collective同样需要这些主题的权限。

节点仅需要对command主题的写权限和对reply主题的读权限。下面的例子同样给了它们管理员权限这样可以动态创建这些主题。为了简化使用通配符来匹配代理名称，可以进一步限制特定节点仅运行特定代理。加上这些限制意味着任何在你的节点上的将无法向command主题进行写操作，这样就不会向剩余的collective发送命令。

对于注册主题有一个特殊情形，如果想启用注册功能，就需要给这个节点command写权限来注册这个代理。在registration主题不需要返回任何东西，这可以在ActiveMQ配置文件中限制。

我们会让mcollective作为mcollective user来登录，创建一个组叫做mcollectiveusers，然后给这个组在启用mcollective的节点上运行特殊注册的权限。

rip user就是mcollective admin而且能够创建命令和接收回复。

先是创建用户和组：

    <simpleAuthenticationPlugin>
         <users>
          <authenticationUser username="mcollective" password="pI1SkjRi" groups="mcollectiveusers,everyone"/>
          <authenticationUser username="rip" password="foobarbaz" groups="admins,everyone"/>
         </users>
        </simpleAuthenticationPlugin>

接着创建访问权限：

    <authorizationPlugin>
      <map>
        <authorizationMap>
          <authorizationEntries>
            <authorizationEntry queue="mcollective.>" write="admins" read="admins" admin="admins" />
            <authorizationEntry topic="mcollective.>" write="admins" read="admins" admin="admins" />
            <authorizationEntry topic="mcollective.*.reply" write="mcollectiveusers" admin="mcollectiveusers" />
            <authorizationEntry topic="mcollective.registration.command" write="mcollectiveusers" read="mcollectiveusers" admin="mcollectiveusers" />
            <authorizationEntry topic="mcollective.*.command" read="mcollectiveusers" admin="mcollectiveusers" />
            <authorizationEntry topic="ActiveMQ.Advisory.>" read="everyone,all" write="everyone,all" admin="everyone,all"/>
          </authorizationEntries>
        </authorizationMap>
      </map>
    </authorizationPlugin>

可以指定具体的节点mcollective.registration.command权限来运行注册代理以保证节点的注册安全。

最后节点需要配置，在server.cfg中至少有以下几项：

    plugin.stomp.user = mcollective
    plugin.stomp.password = pI1SkjRi
    plugin.psk = aBieveenshedeineeceezaeheer

对于客户端，建议用户详细信息配置使用shell环境变量：

    export STOMP_USER=rip
    export STOMP_PASSWORD=foobarbaz
    export STOMP_SERVER=stomp1
    export MCOLLECTIVE_PSK=aBieveenshedeineeceezaeheer

最后当rip user登录到一个拥有这些环境变量的shell时会获得不同命令的所有权限。可以给不同用户整个collective的权限或者通过限制command主题的访问权限来给某个admin用户仅仅执行特定代理的权限。下一步将是设置节点和中间件的TLS。


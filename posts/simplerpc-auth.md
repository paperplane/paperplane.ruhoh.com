---
title: SimpleRPC Authorization
date: '2013-03-15'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','Authorization','SimpleRPC']
---

##SimpleRPC授权##

作为SimpleRPC框架的一部分，授权系统保证了对代理和动作的细粒度控制。结合连接安全性、集中化审计以及加密签名消息，这一系列重要的功能组合允许大公司对MCollective集群进行精确控制。

客户端在请求中将会包括运行客户端进程的UID而授权函数能够获得这个请求的访问权限。

***

####编写授权插件####

编写授权插件很容易，下面的例子允许将只允许RPC调用UID=500，这是一个称为Action Policy的示例插件。

    module MCollective::Agent
        class Service<RPC::Agent
            authorized_by :authorize_it

            # ...
        end
    end

这个类的任何异常都会导致消息不会被处理或者审计。通过把这些类放在libdir路径下的Util目录下来安装这些插件。在代理中使用：

    module MCollective::Agent
        class Service<RPC::Agent
            authorized_by :authorize_it
            # ...
        end
    end

调用authorized_by :authorize_it将会告诉代理使用MCollective::Util::AuthorizeIt类来做授权。

####启用全局RPC审计####

可以在server.cfg中针对所有代理启用某一特定插件，如果这样做了而某个代理又指定有特定插件，则代理将会优先考虑指定的授权。

    rpcauthorization = yes
    rpcauthprovider = action_policy

如果设置rpcauthorization = no并不会禁用所有的代理的授权插件，注意这是个全局操作，针对某个代理特定的授权并没有因此而禁用。


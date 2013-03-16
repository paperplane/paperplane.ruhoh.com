---
title: SSH behavior
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##SSH行为--SSH behavior##

Fabric现在是利用pure-python SSH重新实现连接管理，这意味着有些时候fabric也会受到这个库的局限。fabric既利用了很多SSH的特性，在有些地方同SSH命令行程序相比也不尽相同或者说灵活。

***

####利用原生SSH配置文件####

Fabric的实现允许加载用户SSH配置文件中的部分选项，这个行为默认是未被启用的。如果启用，设置env.use_ssh_config = True，此时下面的这些SSH配置会被Fabric加载和使用：

[1]User和Port将会被作为合适的连接参数被使用，如果Fabric没有专门指定。这是User/Port选择规则是：

如果这些env变量未被指定，全局指定User/Port会替代当前默认（本地用户名和22）

当然，如果env.user/env.port设定，则会覆盖全局指定

在host_string中指定的User/Port会覆盖之前一切，包括ssh_config的值

[2]HostName 能够被用来替换所给定的hostname,如同正常的SSH一样。

[3]IdentityFile 将会被添加而不是替换到env.key_filename

[4]ForwardAgent 将会使env.forward_agent处于一个或的状态，如果任意一个设置为positive值，则agent forwarding被启用。

***

####Unknown Hosts####

SSH对unkown hosts的三种处理方式（reject,add,ask）中，fabric并未实现ask,要么reject，要么add,基于安全和方便的权衡，fabric默认env.reject_unkown_hosts=False。可以根据实际情况改变。

Known Host with Changed Keys

对于Known Hosts 改变了keys时，SSH默认行为或者说是其Python实现是立即放弃连接。

而fabric的做法是在这之上加了env.disable_known_hosts=False，保留默认SSH行为，若设置为True，此时fabric将不加载known_hosts文件，这样在比较时就不会有问题。

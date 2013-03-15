---
title: Write SimpleRPC Client
date: '2013-03-15'
description: 
categories: ['Mcollective','OPS']
tags: ['Mcollective','SimpleRPC']
---

##编写SimpleRPC客户端##

正如在SimpleRPC中所指出一样，可以使用mco rpc命令行工具来调用代理并让它们尽力以合理方式显示结果。如果这些不够可以写自己的代理。

如果你能坚持SimpleRPC的规则，SimpleRPC客户端能够做正常客户端所做的大多数事情并能使其变得很简单。

***

####基本客户端####

客户端大多数都是一个在代码中采用Ruby Mixin的帮助函数的分支，它提供以下：

    准命令行选项来解析帮助输出
    能够添加命令行选项
    能够访问到代理和它的动作
    帮助输出结果的工具
    输出统计的工具
    构建自己的过滤器的工具
    在保留MCollective::Client功能的前提下附加其他功能
    能够很简单也能很复杂

***

####简单客户端示例####

下面是一个为之前Helloworld代理所写的客户端，它调用helloworld代理并打印结果。

    #!/usr/bin/ruby
    require 'mcollective'
    include MCollective::RPC
    mc = rpcclient("helloworld")
    printrpc mc.echo(:msg => "Welcome")
    printrpcstats
    mc.disconnect

将这个文件以hello.rb保存在libdir/mcollective/application/目录下，以mco hello执行即可。

    [root@master application]# mco hello
     * [ ============================================================> ] 1 / 1

    Finished processing 1 / 1 hosts in 61.16 ms

这个执行之前说过正常比如说在1000台机器上它只会返回执行的进度和错误信息，如果想看到正确执行信息只需-v即可。

下面对上面的代码逐行解释：

    include MCollective::RPC
    mc = rpcclient("helloworld")

第一行很熟系的include很多帮助函数，我在查看源码时看到关于MCollective::RPC的介绍，这是一个使用RPC框架创建客户端和代理的标准接口工具集，标准兼容代理能够很容易创建像web 接口这样的通用客户端。

代码首先加载了很多其他模块，这些则是具体功能模块

    autoload :Client, "mcollective/rpc/client"
    autoload :Agent, "mcollective/rpc/agent"
    autoload :Reply, "mcollective/rpc/reply"
    autoload :Request, "mcollective/rpc/request"
    autoload :Audit, "mcollective/rpc/audit"
    autoload :Progress, "mcollective/rpc/progress"
    autoload :Stats, "mcollective/rpc/stats"
    autoload :DDL, "mcollective/rpc/ddl"
    autoload :Result, "mcollective/rpc/result"
    autoload :Helpers, "mcollective/rpc/helpers"
    autoload :ActionRunner, "mcollective/rpc/actionrunner"

另外还有几个函数是我们写客户端需要用到的函数：

    rpcoptions,rpcclient,printrpcstats,printrpc,empty_filter?,request,reply

第二行就为代理'helloworld'创建了新的client,之后你可已通过mc变量来访问。

    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")
    printrpcstats

先是使用mc.echo调用echo这个特定动作，给它传入：msg的参数并将它返回出来。这个参数会随着不同动作而不同，可以看具体的源码。它返回一个简单的数组作为结果。可以以任意形式打印，在后面会讨论。

printrpc，printrpcstats是用于分别打印结果和统计的函数。

    mc.disconnect

将客户端从中间件断开连接。如果不这样像ACtiveMQ这样的中间件会记录异常日志。

***

####改变输出####

输出冗余结果（详细信息）

    mc = rpcclient("helloworld")

    mc.discover :verbose => true

    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC"), :verbose => true

前面看到如果正常执行，则没有任何输出。但如果加了verbose标志后，不需使用-v选项也能输出所有结果：

    [root@master application]# mco hello 
    Determining the amount of hosts matching filter for 2 seconds .... 1

     * [ ============================================================> ] 1 / 1


    master.example.com                      : OK
        {:msg=>"Welcome", :out=>nil, :err=>""}
    ---- rpc stats ----
               Nodes: 1 / 1
         Pass / Fail: 1 / 0
          Start Time: Wed Jul 25 11:13:26 -0400 2012
      Discovery Time: 2003.30ms
          Agent Time: 119.64ms
      Total Time: 2122.94ms

如果你想与所有的客户端交互都输出详细信息，可以：

    mc = rpcclient("helloworld")
    mc.verbose = true

    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

禁用进度条

    mc = rpcclient("helloworld")
    mc.progress = false

默认会动态显示多台机器执行进度，如果想禁用可以如上操作。

结果保存变量而不打印

    stats = mc.echo(:msg => "Welcome to MCollective Simple RPC").stats

    report = stats.report

这样标准输出就不会有，但你同样可以查看到该变量的值。

***

####可编程应用过滤器####

不仅可以在命令行传入--with-*参数来做为过滤，也可以直接通过编程来实现：
    mc = rpcclient("helloworld")

    mc.class_filter /dev_server/
    mc.fact_filter "country", "uk"

    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

还可以设置其他过滤器：agent_filter, identity_filter 和 compound_filter

fact_filter还支持其他几种形式：

    mc.fact_filter "country=uk"
    mc.fact_filter "physicalprocessorcount", "4", ">="

这会限制所有的机器都在UK而且不能超过3个处理器

<strong>重置过滤器为空</strong>

    mc = rpcclient("helloworld")

    mc.class_filter /dev_server/

    mc.reset_filter
    
这样所有被包括的代理的过滤器都被重置为空。

***

####批量处理代理####

默认情况下客户端会同时与所有机器通信，这在有些情况下可能不是你所想的，比如处理DOS或相关组件。

可以指定客户端以批量的形式来与远程代理交互，在每个批量处理过程中sleep。

客户端都有--batch和--batch-sleep-time命令行选项在程序中也可以编程实现：

    mc = rpcclient("helloworld")
    mc.batch_size = 10
    mc.batch_sleep_time = 5

    mc.echo(:msg => "hello world")

或者

    mc = rpcclient("helloworld")

    mc.echo(:msg => "hello world", :batch_size => 10, :batch_sleep_time => 5)

***

####强制重新发现####

默认情况下一个脚本只能执行一次discovery，并重用发现结果。可以通过编程实现使用不同的filter时重新发现：

    mc = rpcclient("helloworld")

    mc.class_filter /dev_server/
    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

    mc.reset

    mc.fact_filter "country", "uk"
    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")
    
这里执行一次mco echo调用会进行两次discovery，并且都是根据不同过滤条件来执行discovery。

***

####应用自己的发现信息####

新的消息支持直接消息模式，这时候可以使用自己的发现信息。这用于比如你在向集群部署某个应用时，你知道某个已知的异常在一些机器上会发生，想更快的确定是那个机器发生问题时可这么做，此外经常还设置TTL。

    mc = rpcclient("helloworld")
    mc.ttl = 3600

    mc.discover(:nodes => ["host1", "host2", "host3"]
    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

***

####仅向部分发现机器发送请求####

默认情况是向所有执行发现之后的机器发送请求。但是如果想把MCollective看成HA服务一样，你可已只针对其中的某些发送请求包括随机与不随机。

前10%选择时：

    mc = rpcclient("helloworld")

    mc.limit_targets = "10%"
    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

随机选择时：

    mc = rpcclient("helloworld")

    mc.limit_targets = "10%"
    mc.limit_method = :random
    printrpc mc.echo(:msg => "Welcome to MCollective Simple RPC")

***

####直接处理结果####

结果与异常对应表：

    状态码  描述    异常类
    0   OK  
    1   OK failed(所有数据解析正确，有某个动作未执行完) RPCAborted
    2   Unknown action 未知操作 UnknownRPCAction
    3   Missing data 缺失数据   MissingRPCData
    4   Invalid data 无效数据   InvalidRPCData
    5   Other error 其他错误    UnknownRPCError

<strong>SimpleRPC风格结果</strong>

SimpleRPC提供由基本客户端库结果修剪的的输出版本，你可以根据需要选择需要返回的结果。这仅仅是ruby语言提供的功能。可以自定义为以下格式：

    mc.echo(:msg => "hello world").each do |resp|
       printf("%-40s: %s\n", resp[:sender], resp[:data][:msg])
    end
    
输出就像:

    dev1.you.net                          : hello world
    dev2.you.net                          : hello world
    dev3.you.net                          : hello world

echo循环结果数组，并选择需要输出的两项打印，结果是一个哈希数组。

    [{:statusmsg=>"OK",
     :sender=>"dev1.your.net",
     :data=>{:msg => "hello world"},
     :statuscode=>0},
    {:statusmsg=>"OK",
     :sender=>"dev2.your.net",
     :data=>{:msg => "hello world"},
     :statuscode=>0}]

这里的statuscode与上表中的返回码对应。

<strong>获取MCollective::Client访问权限#req 结果</strong>

可以实时获取每个结果，这时需要处理异常，这会得到一个不同风格的结果集，这个结果集是所有客户端提供的结果的全部。

    mc.echo(:msg => "hello world") do |resp|
       begin
          printf("%-40s: %s\n", resp[:senderid], resp[:body][:data])
       rescue RPCError => e
          puts "The RPC agent returned an error: #{e}"
       end
    end

打印的结果与之前一样，但是结果集却不同。

    {:msgtarget=>"/topic/mcollective.helloworld.reply",
     :senderid=>"dev2.your.net",
     :msgtime=>1261696663,
     :hash=>"2d37daf690c4bcef5b5380b1e0c55f0c",
     :body=>{:statusmsg=>"OK", :statuscode=>0, :data=>{:msg => "hello world"}},
     :requestid=>"2884afb0b52cb38ea4d4a3146d18ef5f",
     :senderagent=>"helloworld"}

当然也可以获取SimpleRPC风格结果而显示原生客户端结果集。

    mc.echo(:msg => "hello world") do |resp, simpleresp|
       begin
          printf("%-40s: %s\n", simpleresp[:sender], simpleresp[:data][:msg])
       rescue RPCError => e
          puts "The RPC agent returned an error: #{e}"
       end
    end

***

####添加自定义命令行选项####

    #!/usr/bin/ruby

    require 'mcollective'

    include MCollective::RPC

    options = rpcoptions do |parser, options|
       parser.define_head "Generic Echo Client"
       parser.banner = "Usage: hello [options] [filters] --msg MSG"

       parser.on('-m', '--msg MSG', 'Message to pass') do |v|
          options[:msg] = v
       end
    end

    unless options.include?(:msg)
       puts("You need to specify a message with --msg")
       exit! 1
    end

    mc = rpcclient("helloworld", :options => options)

    mc.echo(:msg => options[:msg]).each do |resp|
       printf("%-40s: %s\n", resp[:sender], resp[:data][:msg])
    end

执行结果：

    [root@master application]# mco hello --msg "we"

     * [ ============================================================> ] 1 / 1

    master.example.com                      : we

并且帮助信息：

    [root@master application]# mco hello --help
    Usage: hello [options] [filters] --msg MSG
    Generic Echo Client
        -m, --msg MSG                    Message to pass

            --np, --no-progress          Do not show the progress bar
        -1, --one                        Send request to only one discovered nodes
            --batch SIZE                 Do requests in batches
    ......

此外客户端还支持以下功能：

禁用命令行解析提供默认选项

向SimpleRPC发送请求不需发现和阻止

使用自定义发现策略（discovery）

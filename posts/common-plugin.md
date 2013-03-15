---
title: Common Plugin
date: '2013-03-16'
description: 
categories: ['Mcollective','OPS']
tags: ['Mcollective','Plugin']
---

##常见插件类型##

常见插件类型有以下：data  plugins(数据插件), discovery  plugins(发现插件), result aggregation plugins(结果聚类插件), registration plugins(注册插件), fact source plugins(事实源插件).

***

####data plugins（数据插件）####

直到mcollective 2.0发现系统仍只能发现安装的代理、配置管理类（puppet classes）、facts和节点identities。可以通过插件系统来支持发现更多的源sources.

注意：这个功能在2.1中可用。

基本的思想就是你可以通过下面这样的发现语句来实现：

    % mco find -S "fstat('/etc/rsyslog.conf').md5=/4edff591f6e38/"
    % mco find -S "sysctl('net.ipv4.conf.all.forwarding').value=1"
    % mco find -S "sysctl('net.ipv4.conf.all.forwarding').value=1 and % location=dc1"

也可以在代理或者其他插件中使用这些数据源：

    action "query" do
       reply[:value] = Data.sysctl(request[:sysctl_name]).value
    end

这些数据源作为插件这样就可以通过插件系统来提供并且它们需要DDL文件，DDL文件在客户端和server端都需要使用用以提供验证和配置。

这些插件客户端的DDL作用：

    使用未知函数进行发现时会发生错误
    输入的参数值会由DDL文件进行验证
    只能使用DDL中已知的输出属性
    若插件的DDL文件需要说5秒钟执行发现那么每次执行最大时间自动设为5秒
    server端DDL作用：
    用来验证已知插件
    用来验证输入参数
    用来验证请求的输出值
    查看和检索数据插件结果

可以通过使用rpcutil代理查看和检索数据插件输出：

    % mco rpc rpcutil get_data source=fstat query=/etc/hosts
    ……
    your.node.net
               atime: 2012-06-14 21:41:54
           atime_age: 54128
       atime_seconds: 1339706514
               ctime: 2012-01-18 20:28:34
           ctime_age: 12842128
       ctime_seconds: 1326918514
                 gid: 0
                 md5: 54fb6627dbaa37721048e4549db3224d
                mode: 100644
               mtime: 2010-01-12 13:28:22
           mtime_age: 76457740
       mtime_seconds: 1263302902
                name: /etc/hosts
              output: present
             present: 1
                size: 158
                type: file
                 uid: 0

#####编写数据插件#####

<strong>插件的ruby逻辑</strong>

数据插件任何时候不能改变系统，需要注意创建插件只能读系统状态。如果想改变系统状态就需要编写代理。

这些插件需要保持简单因为它们需要在命令行键入，它们有以下要求：

只能由1个输入参数

返回简单的string，numeric或者boolean，不能有哈希或者负责数据类型

需要执行快速因为这些会影响发现时间和代理运行次数

编写数据插件就像编写代理一样简单，下面是sysctl插件的例子：

    module MCollective
      module Data
        class Sysctl_data<Base
          activate_when { File.executable?("/sbin/sysctl") && Facter["kernel"] == "Linux" }
          query do |sysctl|
            shell = Shell.new("/sbin/sysctl %s" % sysctl)
            shell.runcommand
             if shell.status.exitstatus == 0
               value = shell.stdout.chomp.split(/\s*=\s*/)[1]

               if value
                 value = Integer(value) if value =~ /^\d+$/
                 value = Float(value) if value =~ /^\d+\.\d+$/
               end

               result[:value] = value
             end
           end
         end
       end
     end

类名必须是xxx_data而且必须继承Base。文件需要保存在libdir下以data/sysctl_data.rb和data/sysstl_data.ddl形式。

插件只有在文件/sbin/sysctl存在而且可执行，系统是linux的情况下才会被激活。如果这些代理在windows上将会被禁用因为这些机器使用这个函数不会被发现。

接着创建查询主体部分，使用MCollective：：Shell类来运行sysctl,将结果存入一个result哈希中。

result哈希是这些插件返回值的唯一方式，只能在结果中保存简单的string，number或者boolean。

<strong>插件的DDL文件</strong>

正如前面提到的数据插件需要提供DDL文件，这些文件可模仿SimpleRPC 代理：

    metadata    :name        => "Sysctl values",
                :description => "Retrieve values for a given sysctl",
                :author      => "R.I.Pienaar <rip@devco.net>",
                :license     => "ASL 2.0",
                :version     => "1.0",
                :url         => "http://marionette-collective.org/",
                :timeout     => 1
    dataquery :description => "Sysctl values" do
         input :query,
               :prompt => "Variable Name",
               :description => "Valid Variable Name",
               :type => :string,
               :validation => /^[\w\-\.]+$/,
               :maxlength => 120

         output :value,
                :description => "Kernel Parameter Value",
                :display_as => "Value"
     end

超时（timeout）必须被正确设置，如果数据源很慢需要在超时这反应出来。客户端根据超时值来决定时候等待网络结果返回，因此如果返回错误会导致节点未被发现。

每个数据插件只能由1个数据查询块和唯一的一个输入块但是可以有多个输出块。

验证正确很重要，这里仅接受字符很容易知道这在linux sysctl变量是合法的。将避免使用引号来避免意外。

这里需要注意的是输出的名字和在discovery和代理中使用时的关联性。这里创建了一个叫做value的输出，那么意味着在发现时可以这样写：

    % mco find -S "sysctl('net.ipv4.conf.all.forwarding').value=1"

在插件输出结果时这样写：

    1 result[:value] = value

在任意使用这个数据源的代理中这样写：
    
    something = Data.sysctl('net.ipv4.conf.all.forwarding').value

这些在很多地方都必须一致，不能引用未定义的数据而且也不能使用未经DDL声明的验证规则验证的输入。

同样和其他插件一样可以通过mco plugin doc sysctl自动生成文档、查看帮助信息。

通过mco inventory nodename来查看可用插件。

***

####discovery plugins(发现插件)####

直到MCollective 2.0.0 发现系统还只能通过中间件广播来发现网络。

2.0引进的直接寻址性能将使用户能够不使用广播而同节点通信，如果他知道这个节点的identity。

在2.1版本中介绍了新的插件，能够在任意数据源是进行发现并且返回一系列identities。比如flatfiles 或者databases。

关于直接寻址功能在编写代理的时候有简单提到。这一部分在目前稳定版2.0中并未太多应用。一般直接使用默认发现的广播方式。

***

####aggregate plugins(结果聚类插件)####

MCollective 代理返回数据时尽可能提供更可用的用户接口。为了这一点要求代理拥有DDL文件来描述返回的数据。

DDL文件用来配置客户端，也和帮助生成用户接口相关。它们用来提醒作为动作需要参数和当回复来到时呈现结果。例如将:freecpu在展现的时候变成“Free CPU”。

之前如果代理返回数据需要做统计的时候就需要另外写自定义应用程序。这是来自mco nrpc的例子：

    % mco nrpe check_load
    ……
    Finished processing 25 / 25 hosts in 556.48 ms

                  OK: 25
             WARNING: 0
            CRITICAL: 0
             UNKNOWN: 0

这里显示统计结果的是一个自定义的插件，而其他使用其他客户端想与代理进行交互的应用程序则无法有这个总结。

通过使用聚类插件和更新DDL可以在所有的结果集中提供这样的统计结果并且通过mco rpc应用程序展现和任何调用printrpc。

    % mco rpc nrpe runcommand command=check_load
    Discovering hosts using the mongo method .... 25
    * [============================================================> ] 25 / 25

    Summary of Exit Code:

                OK : 25
           WARNING : 0
           UNKNOWN : 0
          CRITICAL : 0


    Finished processing 25 / 25 hosts in 390.70 ms

这里显示之前类似的统计结果，所有的这些只需通过一个简单的聚类插件实现，这个插件为客户端而写并随客户端部署。

上面显示的结果是通过使用printrpcstats实现，也可以获取原始数据并进一步选择呈现方式--或许可以在web 接口使用图像来展现。

社区提供了很多聚类插件而且可以写很多自定义的插件。在原始安装的rpcutil代理的collective_info、get_fact、daemon_stats，get_config_item动作中都有这些统计的应用。

#####使用现有插件#####

<strong>更新DDL文件</strong>

目前MCollective提供三个插件agverage(),summary()和sum()，可以在任何一个代理中使用作为统计结果，这是一个在rpcutil代理DDL文件中的使用示例：

    action "get_config_item", :description => "Get the active value of a specific config property" do
        output :value,
               :description => "The value that is in use",
               :display_as => "Value"

        summarize do
            aggregate summary(:value)
        end
    end

这里注意到输出数据的结果称为:value，在输出summary函数的引用中使用summary(:value)。结果如下所示:

    % mco rpc rpcutil get_config_item item=collectives
    ……
    dev8
       Property: collectives
          Value: ["mcollective", "uk_collective"]

    Summary of Value:

          mcollective = 25
        uk_collective = 15
        fr_collective = 9
        us_collective = 1

    Finished processing 25 / 25 hosts in 349.70 ms

summary()函数为最终数据分布显示提供了表格的形式。

<strong>获取原始统计结果</strong>

如果想做一些比如制作一张整个结果的图像或者在一个web页面展示统计信息，就需要获取结果的原始数据，这是一些获取所有计算统计值得ruby代码：

    require 'mcollective'

    include MCollective::RPC

    c = rpcclient("rpcutil")
    c.progress = false

    c.get_config_item(:item => "collectives")

    c.stats.aggregate_summary.each do |summary|
      puts "Summary of type: %s" % summary.result_type
      puts "Display format: '%s'" % summary.aggregate_format
      puts
      pp summary.result
    end

正如你所看见的会得到统计数组，这是因为每个DDL会使用多次聚类调用，会是所有计算结果的数组：

    Summary of type: collection
    Display format: '%13s = %s'

    {:type=>:collection,
    :value=>
      {"mcollective"=>25,
       "fr_collective"=>9,
       "us_collective"=>1,
       "uk_collective"=>15},
    :output=>:value}

这有两种类型的结果:collective和:numeric,就numeric结果而言:value恰好是数字。

aggregate_format或者是用户提供的形式或者是动态计算的形式用来在console上展示统计结果。在这种情况下每对哈希都将以合理正确的键值对形式呈现。

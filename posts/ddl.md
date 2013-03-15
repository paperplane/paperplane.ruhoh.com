---
title: DDL
date: '2013-03-15'
description:
categories: ['Mcollective','OPS']
tags: Mcollective
---

##DDL数据定义语言##

同其他远程过程调用系统，MCollective有一个DDL来定义可用远程方法以及需要的输入和产生的输出。

除了这些过程定义同样提供关于作者，版本等其他关键数据点的元数据。

DDL在以下场景使用：

    作为用户帮助页面来使用

    作为自动生成用户接口的方式

    RPC客户端在等待回复时自动配置合理的超时时间

    在发送之前验证网络输入以避免向远程节点发送不必要数据

    模块仓库可以使用元数据来展示可用模块的标准视图帮助用户选择合适模块

    server将在发送给代理之前验证传入的请求

***

下面会以安装过的server代理来做说明。

####元数据（metadata）####

首先定义代理元数据，包括名称、描述、作者、许可证、版本、超时等信息。

    metadata :name        => "SimpleRPC Service Agent",
             :description => "Agent to manage services using the Puppet service provider",
             :author      => "R.I.Pienaar",
             :license     => "GPLv2",
             :version     => "1.1",
             :url         => "http://projects.puppetlabs.com/projects/mcollective-plugins/wiki",
             :timeout     => 60
    
这些作用都很明显。:timout是MCollective 守护进程允许线程执行的时间。

动作、输入和输出（actions,input and output）

定义输入输出是最复杂的部分，下面是status 动作：

    action "status", :description => "Gets the status of a service" do
        display :always  # supported in 0.4.7 and newer only
        input :service,
              :prompt      => "Service Name",
              :description => "The service to get the status for",
              :type        => :string,
              :validation  => '^[a-zA-Z\-_\d]+$',
              :optional    => false,
              :maxlength   => 30
         output :status,
                :description => "The status of service",
                :display_as  => "Service Status",
                :default     => "unknown status"
     end
    
正如上面所示，可以定义输入输出参数所有重要组件。：type就是其中拥有不同值的参数之一。在2.1.1.中，输出能够定义默认值，这样代理的回复结构就会预先填充定义的输出，如果未提供默认值，就会是nil.

默认情况下mcollective仅仅下士动作失败执行的数据，上面的:display行告诉永远显示结果，可能的值是:ok,:failed(默认行为)和:always。

最后代理还有3个同样的动作，start,stop和restart，下面使用一个循环：

     ["start", "stop", "restart"].each do |act|
        action act, :description => "#{act.capitalize} a service" do
            input :service,
                  :prompt      => "Service Name",
                  :description => "The service to #{act}",
                  :type        => :string,
                  :validation  => '^[a-zA-Z\-_\d]+$',
                  :optional    => false,
                  :maxlength   => 30
             output :status,
                    :description => "The status of service after #{act}",
                    :display_as  => "Service Status",
                    :default     => "unknown status"
         end
     end

所有的这些代码都在一个文件里面，没有特殊的类和模块，仅仅是保存为service.ddl并保存在service.rb同一路径下。

重要的是你可以在使用DDL的时候不用service.rb文件，意味着在运行代理程序机器上你可以仅仅使用.ddl文件放在代理目录下即可。

使用mco plugin do service的输出如下：

    % mco plugin doc service
    SimpleRPC Service Agent
    =======================
    Agent to manage services using the Puppet service provider
          Author: R.I.Pienaar
         Version: 1.1
         License: GPLv2
         Timeout: 60
       Home Page: http://projects.puppetlabs.com/projects/mcollective-plugins/wiki
    ACTIONS:
    ========
       restart, start, status, stop
       restart action:
       ---------------
           Restart a service
           INPUT:
               service:
                  Description: The service to restart
                       Prompt: Service Name
                         Type: string
                   Validation: ^[a-zA-Z\-_\d]+$
                       Length: 30
           OUTPUT:
               status:
                  Description: The status of service after restart
               Display As: Service Status

####可选输入（Optional Inputs）####

再输入块中有一个强制的:optional 字段，当其设置为true时，客户端程序在调用代理时没有提供输入是可以的，如果提供了同样也会验证。

输入类型(Types of Input)

    :type选项可以使:string,:list:boolean或者:any等。

    :string类型
    string类型首先验证事实上是否是字符串，然后验证长度，最后再验证提供的正则表达式。:validation和:maxlength是string 类型输入要求提供的两个参数，如果想允许无限长度文本，可以设置:maxlength=>0但是需要小心。

    :list
    list类型需要提供一系列有效可选参数值，只有这些才是被允许的。如下：

    input :action,
          :prompt      => "Service Action",
          :description => "The action to perform",
          :type        => :list,
          :optional    => false,
          :list        => ["stop", "start", "restart"]
    在用户界面这可以被作为一个下拉菜单或者其他形式菜单来显示。

    :boolean这个值或者是true或者是false

    :integer这个输入值应该是像1或者100这样的数字

    :float这个输入应该是像1.1这样的值而不是1

    :number这里的值既可以是integer也可以是number

    :any这个输入值可以是任何类型，这允许发送像哈希的数组这样丰富的对象，它禁用了输入类型的验证机制。

####获取DDL####

当编写客户端应用程序或者 WEB 应用的时候可以通过多种方式来获取任何代理的DDL访问权限：

    require 'mcollective'
    config = MCollective::Config.instance
    config.loadconfig(options[:config])
    ddl = MCollective::DDL.new("service")
    puts ddl.help("#{config.configdir}/rpc-help.erb")
    这会产生一个文本帮助输出，可以应用任何ERB模板来格式化输出。可以直接获取数据结构：
    ddl = MCollective::DDL.new("service")
    puts "Meta Data:"
    pp ddl.meta
    puts
    puts "Status Action:"
    pp ddl.action_interface("status")
    
结果会是：

    Meta Data:
    {:license=>"GPLv2",
     :author=>"R.I.Pienaar",
     :name=>"SimpleRPC Service Agent",
     :timeout=>60,
     :version=>"1.1",
     :url=>"http://projects.puppetlabs.com/projects/mcollective-plugins/wiki",
     :description=>"Agent to manage services using the Puppet service provider"}
     Status Action:
     {:action=>"status",
      :input=>
       {:service=>
         {:validation=>"^[a-zA-Z\\-_\\d]+$",
          :maxlength=>30,
          :prompt=>"Service Name",
          :type=>:string,
          :optional=>false,
          :description=>"The service to get the status for"}},
      :output=>
       {"status"=>
         {:display_as=>"Service Status", :description=>"The status of service"}},
      :description=>"Gets the status of a service"}


DDL对象可以在任何rpcclient中可用：

    service = rpcclient("service")
    pp service.ddl.meta

在通过service来访问DDL的例子时，如果在service代理的机器上没有DDL文件则会从ddl访问器返回nil。


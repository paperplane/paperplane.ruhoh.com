---
title: Node Report
date: '2013-03-15'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','Report']
---

##自定义节点报告##

正如在概述中介绍一样，MCollective可以用来撰写自定义设施报告。既然我们有节点的facts,classes和agents这些信息，它们都可以用来撰写自定义报告。mco inventory是一个通用的节点和网络报告工具，它还拥有基本的脚本执行功能。这是一项新兴的功能，脚本语言可能会变。

***

####节点预览####

对于给定节点可以运行mco inventory来获取完全信息。如下：
    
    [root@master mcollective]# mco inventory master.example.com
    Inventory for master.example.com:
       Server Statistics:
                          Version: 2.0.0
                       Start Time: Tue Jul 24 16:04:31 -0400 2012
                      Config File: /etc/mcollective/server.cfg
                      Collectives: mcollective, puppet, local
                  Main Collective: mcollective
                       Process ID: 6822
                   Total Messages: 11
          Messages Passed Filters: 11
                Messages Filtered: 0
                 Expired Messages: 0
                     Replies Sent: 10
             Total Processor Time: 0.48 seconds
                      System Time: 0.13 seconds
       Agents:
          discovery       helloworld      package        
          process         rpcutil         service        
       Configuration Management Classes:
          default                        settings                      
       Facts:
          architecture => i386
          augeasversion => 0.10.0
          boardmanufacturer => LENOVO
          boardproductname => 284255C
      boardserialnumber => 1ZG6Y9AT0GJ

####Collective列表####

之前介绍了subcolletives概念，可以使用inventory应用程序来查看所有已知collectives.

    [root@master mcollective]# mco inventory --list-collectives
       Collective                     Nodes
       ==========                     =====
       local                          1
       web                            1
       puppet                         2
       mcollective                    3
                     Total nodes: 3

####Collective图表####

同样可以为所有collectives创建一个dot格式图表.

    [root@master mcollective]# mco inventory --collective-graph collective.dot
    Retrieving collective info....
    Graph of 3 nodes has been written to collective.dot
    graph {
       subgraph "local" {
          "master.example.com" -- "local"
       }
       subgraph "mcollective" {
          "puppet.example.com" -- "mcollective"
          "web.example.com" -- "mcollective"
          "master.example.com" -- "mcollective"
       }
       subgraph "puppet" {
          "puppet.example.com" -- "puppet"
          "master.example.com" -- "puppet"
       }
       subgraph "web" {
          "web.example.com" -- "web"
       }
    }

####自定义报告####

可以创建一些简短的脚本作为--script选项参数传递给mco inventory。

在报告中可以有以下可用数据：

    |  变量  |             描述              |
    |  time  |报告创建时间,通常是ruby时间对象|
    |identity|          senderid             |
    | facts  |        facts的哈希值          |
    | agents |        代理的数组             |
    |classes |       CF Classes的数组        |

<strong>Printf风格报告</strong>

根据文档介绍，将来会添加更多功能。目前为止可以通过hash类型来访问facts，agents和classes视为数组而identity视为string.

    inventory do
        format "%s:\t\t%s\t\t%s"

        fields { [ identity, facts["lsbdistdescription"],facts["ipaddress_eth0"] ] }
    end

输出

    [root@master mcollective]# mco inventory --script inventory.mc  
    puppet.example.com:     CentOS release 5.7 (Final)      10.217.12.211
    master.example.com:     CentOS release 6.2 (Final)      10.217.12.170

<strong>Perl格式风格报告</strong>

使用这种风格需要先安装formatr gem。即：gem install formatr.一旦安装好就可以创建如下脚本：

    formatted_inventory do
        page_length 20
        page_heading <<TOP
                 Node Report @<<<<<<<<<<<<<<<<<<<<<<<<<
                            time
    Hostname:         Puppet Versin:     Distribution:
    -----------------------------------------------------------------------
    TOP
        page_body <<BODY
    @<<<<<<<<<<<<<<<< @<<<<<<<<<<<< @<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
    identity,    facts["puppetversion"], facts["lsbdistdescription"]
                                    @<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
                                    facts["processor0"]
    BODY
    end

这里创建了自定义报告格式为：每页显示20个节点信息，有标题区域和每个节点会有两行报告信息。输出为：

    [root@master mcollective]# mco inventory --script inventory1.mc
                 Node Report Tue Jul 24 16:24:53 -0400
    Hostname:         Puppet Versin:     Distribution:
    -----------------------------------------------------------------------
    master.example.com 2.6.16        CentOS release 6.2 (Final)
                                    Intel(R) Core(TM)2 Duo CPU     T6670  @
    puppet.example.com 2.6.13        CentOS release 5.7 (Final)
                                    Intel(R) Core(TM)2 Duo CPU     T6670  @

这种报告虽然很丑但很简单方便。


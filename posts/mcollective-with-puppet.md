---
title: Mcollective With Puppet
date: '2013-03-16'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','Puppet']
---
##MCollective结合Puppet##

####Puppet给MCollective带来了什么####

增强Puppet与MCollective联系，可以使MCollective更好利用Puppet资源来加快自己发展，并尽可能在将来将MCollective项目集成到Puppet平台。

目前能看见的应用就是Puppet资源作为Facts，Puppet Class作为过滤器。

***

####MCollective给Puppet带来了什么####

MCollective为采用Puppet模型驱动框架来进行系统管理的数千企业带来了多主机、多数据中心编排服务。结合MCollective，Puppet增加了重要的时间元素来进行系统管理。它同时也提高了Puppet管理现有系统上的应用程序的执行策略。MCollective的实时发现性能加上Puppet Dashboard，简化了用户使用Puppet平台数据来进行复杂队列活动的调度。这同时将消息队列与实时发现复杂网络资源相联系。

下面两幅图是是对上面内容的进一步解释。

![图片]({{urls.media}}/mcollective/mco-puppet.png)
![图片]({{urls.media}}/mcollective/puppet-mco.png)

***

####MCollective通过插件管理Puppet####

MCollective为Puppet管理增加时间元素，这应该是MCollective能够管理控制调度Puppet的原因所在。现在社区中已有三个比较成型的MCollective管理Puppet插件。

<strong>Puppetd</strong>
    
该代理能够启用、禁用 或者 客户端强制执行Puppetd。
    
这个代理能够很好实现puppet kick执行，但是使用这个代理有两个问题：1.自身设计问题：Puppet在繁忙的时候处理加锁解锁(这里面的enable/disable)工作的不是很好，有很多bugs。
    
可以通过运行这个插件实现运行正常Puppetd一样。这个代理会发送一个HUP信号如果检测到Puppet正在运行。由于Puppet版本的一个bug(在版本2.7.10和2.6.14中修复)，这个版本的代理会错误地将实际上Puppet Stopped的状态报告为Idling。这是第二个问题。下面是一些使用情况：

在所有机器上执行，查看状态

    [root@master ~]# mco puppetd status
     * [ ============================================================> ] 3 / 3
    master.example.com                       Currently stopped; last completed run 8012969 seconds ago
    web.example.com                          Currently stopped; last completed run 8172598 seconds ago
    puppet.example.com                       Currently stopped; last completed run 8172571 seconds ago

    Finished processing 3 / 3 hosts in 113.60 ms

使用过滤器选择特定主机查看

    [root@master ~]# mco puppetd -I /master/ status
     * [ ============================================================> ] 1 / 1
    master.example.com                       Currently stopped; last completed run 8013826 seconds ago

    Finished processing 1 / 1 hosts in 48.95 ms

    [root@master ~]# mco puppetd -I /example/ status
     * [ ============================================================> ] 3 / 3
    master.example.com                       Currently stopped; last completed run 8013842 seconds ago
    web.example.com                          Currently stopped; last completed run 8173470 seconds ago
    puppet.example.com                       Currently stopped; last completed run 8173406 seconds ago

    Finished processing 3 / 3 hosts in 60.91 ms
    [root@master ~]# mco puppetd -W country=China status
     * [ ============================================================> ] 2 / 2
    master.example.com                       Currently stopped; last completed run 8013877 seconds ago
    puppet.example.com                       Currently stopped; last completed run 8173441 seconds ago

    Finished processing 2 / 2 hosts in 60.78 ms
    
强制执行一次Puppetd

    [root@master ~]# mco puppetd -W country=uk runonce
     * [ ============================================================> ] 1 / 1

    Finished processing 1 / 1 hosts in 2061.79 ms

作为RPC代理执行Puppetd 

    [root@master ~]# mco rpc puppetd status
    Determining the amount of hosts matching filter for 2 seconds .... 3
     * [ =========================================================> ] 3 / 3

    master.example.com                       
         Idling: 0
         Status: stopped
        Stopped: 1
        Running: 0
         Status: Currently stopped; last completed run 8016997 seconds ago
       Last Run: 1335381650
        Enabled: 1

    puppet.example.com                       
         Status: stopped
         Idling: 0
        Stopped: 1
        Running: 0
       Last Run: 1343383501
         Status: Currently stopped; last completed run 552 seconds ago
        Enabled: 1

    web.example.com                          
         Idling: 0
         Status: stopped
        Stopped: 1
        Running: 0
         Status: Currently stopped; last completed run 8176626 seconds ago
       Last Run: 1335207620
        Enabled: 1

    Finished processing 3 / 3 hosts in 145.02 ms

    [root@master ~]# mco puppetd summary -I puppet.example.com
     * [ ============================================================> ] 1 / 1
    puppet.example.com                       
          Events: {"total"=>0}
           Times: {"file"=>0.004714,
                   "config_retrieval"=>0.140156984329224,
                   "last_run"=>1343383501,
                   "filebucket"=>0.000676,
                   "total"=>0.148013984329224,
                   "schedule"=>0.002467}
        Versions: nil
       Resources: {"changed"=>0, "failed"=>0, "out_of_sync"=>0, "total"=>9, "restarted"=>0}
         Changes: {"total"=>0}

    Finished processing 1 / 1 hosts in 67.10 ms

    [root@master ~]# mco puppetd runonce -I puppet.example.com -I web.example.com -v
     * [ ============================================================> ] 2 / 2
    puppet.example.com                      : OK
        {:status=>"idling",     :lastrun=>1343384585,     :enabled=>1,     :running=>0,     :stopped=>0,     :idling=>1,     :output=>      "Signalled daemonized puppet agent to run (process 16257); Currently idling; last completed run 11 seconds ago"}

    web.example.com                         : OK
        {:status=>"idling",     :enabled=>1,     :lastrun=>1335207620,     :running=>0,     :stopped=>0,     :idling=>1,     :output=>      "Called /usr/sbin/puppetd --onetime --splaylimit 100 --splay, Currently idling; last completed run 8177193 seconds ago"}

    ---- rpc stats ----
               Nodes: 2 / 2
         Pass / Fail: 2 / 0
          Start Time: Fri Jul 27 10:26:54 -0400 2012
      Discovery Time: 0.00ms
          Agent Time: 1869.93ms
          Total Time: 1869.93ms
    [root@master ~]# mco puppetd status -I puppet.example.com -I web.example.com -v
     * [ ============================================================> ] 2 / 2
    puppet.example.com                       Currently idling; last completed run 30 seconds ago
    web.example.com                          Currently idling; last completed run 15 seconds ago

    ---- rpc stats ----
               Nodes: 2 / 2
         Pass / Fail: 2 / 0
          Start Time: Fri Jul 27 10:27:28 -0400 2012
      Discovery Time: 0.00ms
          Agent Time: 103.71ms
      Total Time: 103.71ms

在puppet这两个节点只中我定义的是两个文件，执行后检查也成功生成该文件。

统计结果

    [root@master agent]# mco puppetd summary
     * [ ============================================================> ] 3 / 3
    master.example.com                       Unknown Request Status
       can not convert nil into Hash

    puppet.example.com                       
          Events: {"success"=>1, "total"=>1}
           Times: {"file"=>0.013021,
                   "filebucket"=>0.000413,
                   "schedule"=>0.002585,
                   "last_run"=>1343398708,
                   "config_retrieval"=>0.125710964202881,
                   "total"=>0.141729964202881}
        Versions: nil
       Resources: {"changed"=>1, "restarted"=>0, "failed"=>0, "out_of_sync"=>1, "total"=>9}
         Changes: {"total"=>1}

    web.example.com                          
          Events: {"total"=>0}
           Times: {"file"=>0.0014,
                   "filebucket"=>0.000633,
                   "schedule"=>0.003076,
                   "last_run"=>1343399603,
                   "config_retrieval"=>0.120060205459595,
                   "total"=>0.125169205459595}
        Versions: nil
       Resources: {"changed"=>0, "restarted"=>0, "failed"=>0, "out_of_sync"=>0, "total"=>8}
         Changes: {"total"=>0}

    Finished processing 3 / 3 hosts in 90.46 ms

    [root@master agent]# mco puppetd count

              Nodes currently enabled: 3
             Nodes currently disabled: 0
    Nodes currently doing puppet runs: 0
              Nodes currently stopped: 0
               Nodes currently idling: 3

    Finished processing 3 / 3 hosts in 68.22 ms

<strong>Puppet CA</strong>

这个代理允许sign、list、revoke和clean Puppet颁发证书。使用举例：

在master端执行puppetca list

    [root@master ~]# mco rpc puppetca list -I master.example.com
     * [ ========================================================> ] 1 / 1
    master.example.com                       
       Waiting CSRs: []
             Signed: ["client.example.com",
                      "learn.localdomain",
                      "master.example.com",
                      "puppet.example.com",
                      "web.example.com"]

    Finished processing 1 / 1 hosts in 59.78 ms

执行Clean操作

    [root@master ~]# mco rpc puppetca clean certname=master.example.com  -I master.example.com -v
     * [ =====================================================> ] 1 / 1
    master.example.com                      : OK
        {:msg=>      "Removed signed cert: /var/lib/puppet/ssl/ca/signed/master.example.com.pem."}

    ---- puppetca#clean call stats ----
               Nodes: 1 / 1
         Pass / Fail: 1 / 0
          Start Time: Fri Jul 27 13:52:19 -0400 2012
      Discovery Time: 0.00ms
          Agent Time: 87.32ms
          Total Time: 87.32ms

查看Clean结果

    [root@master ~]# mco rpc puppetca list -I master.example.com
     * [ ======================================================> ] 1 / 1
    master.example.com                       
       Waiting CSRs: []
             Signed: ["client.example.com",
                      "learn.localdomain",
                      "puppet.example.com",
                      "web.example.com"]

    Finished processing 1 / 1 hosts in 61.99 ms

<strong>PuppetRAL</strong>

    
这个代理允许你使用通过SimpleRPC任意Puppet提供者。

ral支持search,create,find，先查看资源

    [root@master ~]# mco rpc puppetral search type=host -I master.example.com
     * [ ============================================================> ] 1 / 1
    master.example.com                       
       localhost.localdomain: {"parameters"=>
                                {:provider=>:parsed,
                                 :host_aliases=>["localhost"],
                                 :target=>"/etc/hosts",
                                 :audit=>[:ensure, :ip, :host_aliases, :target],
                                 :ensure=>:present,
                                 :ip=>"127.0.0.1",
                                 :loglevel=>:notice},
                               "exported"=>false,
                               "title"=>"localhost.localdomain",
                               "tags"=>["host", "localhost.localdomain"],
                               "type"=>"Host"}
          master.example.com: {"parameters"=>
                                {:provider=>:parsed,
                                 :host_aliases=>["master", "localhost6.localdomain6", "localhost6"],
                                 :target=>"/etc/hosts",
                                 :audit=>[:ensure, :ip, :host_aliases, :target],
                                 :ensure=>:present,
                                 :ip=>"::1",
                                 :loglevel=>:notice},
                               "exported"=>false,
                               "title"=>"master.example.com",
                               "tags"=>["host", "master.example.com"],
                               "type"=>"Host"}
             web.example.com: {"parameters"=>
                                {:provider=>:parsed,
                                 :host_aliases=>[],
                                 :target=>"/etc/hosts",
                                 :audit=>[:ensure, :ip, :host_aliases, :target],
                                 :ensure=>:present,
                                 :ip=>"10.217.12.213",
                                 :loglevel=>:notice},
                               "exported"=>false,
                               "title"=>"web.example.com",
                               "tags"=>["host", "web.example.com"],
                               "type"=>"Host"}
          puppet.example.com: {"parameters"=>
                                {:provider=>:parsed,
                                 :host_aliases=>[],
                                 :target=>"/etc/hosts",
                                 :audit=>[:ensure, :ip, :host_aliases, :target],
                                 :ensure=>:present,
                                 :ip=>"10.217.12.211",
                                 :loglevel=>:notice},
                               "exported"=>false,
                               "title"=>"puppet.example.com",
                               "tags"=>["host", "puppet.example.com"],
                               "type"=>"Host"}

    Finished processing 1 / 1 hosts in 61.32 ms

然后create资源

    [root@master ~]# mco rpc puppetral create type=host title=test.com
    Determining the amount of hosts matching filter for 2 seconds .... 3
     * [ ============================================================> ] 3 / 3
    master.example.com                       
         Status: Resource was created
       Resource: {"tags"=>["host", "test.com"],
                  "type"=>"Host",
                  "parameters"=>
                   {:provider=>:parsed,
                    :host_aliases=>:absent,
                    :target=>:absent,
                    :audit=>[:ensure, :ip, :host_aliases, :target],
                    :ensure=>:absent,
                    :loglevel=>:notice,
                    :ip=>:absent},
                  "title"=>"test.com",
                  "exported"=>false}
    .......
    .......
    Finished processing 3 / 3 hosts in 149.45 m

再find刚才创建的资源

    [root@master ~]# mco rpc puppetral find type=host title=test.com -I master.example.com
     * [ ============================================================> ] 1 / 1
    master.example.com                       
             tags: ["host", "test.com"]
             type: Host
          managed: unknown
            title: test.com
       parameters: {:ip=>:absent,
                    :provider=>:parsed,
                    :target=>:absent,
                    :host_aliases=>:absent,
                    :loglevel=>:notice,
                    :audit=>[:ensure, :ip, :host_aliases, :target],
                    :ensure=>:absent}
         exported: false

    Finished processing 1 / 1 hosts in 67.59 ms

***

####Puppet作为MCollective过滤器####

<strong>使用facts和class过滤器</strong>

社区已有现成插件能让我们启用facter作为fact source。只需要安装完该插件后将配置文件中相应项改为：factsource=>facter(一般默认是factsource=>yaml)。这样就能使用facter提供的facts作为过滤器，在自定义报告中也可以使用。

    $ mco find --with-fact lsbdistrelease=5.4

通常还有另外一种方式就是将通过Puppet，facter中的内容写入facts.yaml文件。这个在之前有介绍，一般通过以下脚本即可实现。这有个好处就是我们依然可以使用yaml文件作为factsource，可以很方便在其中定义自定义的一些facts，比如定义每个server的地点、角色等信息，这些也是很好的过滤器信息。

    package {
    "mcollective": ensure => "0.4.10-1";
    ...
    }

    file {
    "/etc/mcollective/facts.yaml":
      ensure => file,
      content => inline_template("<%= scope.to_hash.reject { |k,v| !( k.is_a?(String) && v.is_a?(String) ) }.to_yaml %>"),
      require => Package["mcollective"];
    }

Puppet为每个node提供了一系列classes,这些classes默认定义在文件(版本不同可能路径不同)/var/lib/puppet/state/classes.txt中。可以通过--with-class过滤器使用这些classes。只需要在每个节点的server.cfg配置文件中加入：

    classesfile = /var/lib/puppet/state/classes.txt

然后就能正常使用--with-class了：

    $ mco find --with-class /apache/

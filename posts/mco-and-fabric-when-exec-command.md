---
title: Mcollective/Fabric Exec Commands
date: '2013-03-16'
description:
categories: ['Mcollective','OPS']
tags: ['Mcollective','Fabric']
---

##MCollective/Fabric执行任务比较##

####定义主机####

<strong>发现主机</strong>
MCollective有丰富的策略来发现主机，目前支持的有通过主机标识（identity，一般为主名）、puppet class、facts、安装的插件以及自定义数据源和自定义发现策略，很灵活。

Fabric仅仅使用主机名来定义主机，使用传统的SSH循环，关于这点比较在最开始已经提到过。

关键一点：MCollective对于需要执行任务的主机采用的主动发现的方式，而Fabric则是“被动”指出。

<strong>定义主机</strong>

Fabric有Host和Role关键字来指定，既可以单独制定某一台，也可以是某一个类别。

MCollective没有Role的概念，但是可以通过多种手段实现Role的功能：通过class分类、直接在fact中添加role=>puppet这样。而且方式更灵活。与Puppet结合后能够查询Puppet资源，这可以为指定在哪些主机上提供参考。

***

####指定列表####

Fabric可以通过Host和Role相结合以多种方式来指定主机。并且会对这些列表中重复主机自动合并和排除其中某个或某些特定主机。

MCollective在制定列表时使用内建的过滤机制，即通过发现主机的策略来过滤主机。现在在过滤过程中已支持很丰富的过滤方法，包括对正则表达式、多种逻辑运算符和算数运算符的支持。它的一个“缺陷”是没有排除主机的关键字，必须通过改变过滤内容来实现，只是做法需要换一种策略。

***

####定义任务####

对于Fabric定义任务，本身执行shell命令很简单，只需调用Fabric相应API来执行就是，但是Fabric定义任务时需要考虑很多，包括执行策略、执行方式这些都影响任务定义。在 Fabric中有API local/run/sudo,这些命令调用的本质都是Subprocess模块。

对于MCollective执行Shell命令，也是调用Shell类的相应接口，在Shell类中有runcommand函数用于执行命令，该函数本质上是调用systemu, 这个ruby库的特点就是很好的处理标准输出与标准错误和子进程僵死问题。如果是用MCollective定义任务不用判断是本地还是远程连接（一般都是远程），默认就是并行执行。MCollective支持对任务的“封装”“重用”，比如安装程序可以定义一个代理就负责安装，在安装不同软件时可以重用这个代理，只是传入的安装程序参数不同。

根据需求，在定义任务时需要制定任务黑名单列表。在使用Fabric时这个就完全交给应用程序自身处理，完全是python逻辑处理，不需要由Fabric这边判断，它只是对判断可以执行的命令进行执行。对于MCollective，可用输入验证来做，目前验证支持类型验证（即基本ruby数据类型），正则表达式验证，或者不设验证。

Fabric在远程交互时需要解决两个很重要的问题，一个涉及到合并和分开输出错误流，一个涉及到什么时候激活prompts等待用户输入。MCollective输出错误已由systemu模块处理，由于采用中间件连接没有密码这类提示问题。

***

####并行执行####

<strong>执行方式</strong>

Fabric默认执行方式是但任务单主机串行方式，MCollective原生支持但任务多主机并行执行方式。二者的并行本质上都是单任务多主机的并行执行，但是任务的定义都很灵活，即任务内容可以很丰富。

<strong>执行效果</strong>

这个效果需要做测试。从建立连接来看，Fabric采用原始SSH连接，MCollective采用中间件和消息队列，抛弃SSH 的For循环。关于与SSH循环的比较开始有详细比较。优势简单说就是异步/事件驱动，自动发现主机，模块化插件等。但是这对发现系统的依赖很大，受网络影响也很大，如果发现过程延迟很大，会直接导致未发现任何节点信息。

***

####输出比较####

MCollective 在输出上花费了很大心血，既有针对每次执行每个主机单个的输出，又有对整体执行情况统计的输出。而且这个输出可以很好地制作统计表格图像和web接口。Fabric输出针对串行执行时是一个个任务一个个主机输出，而在并行执行时，针对一个任务，多个主机则是并行输出，虽然每行输出之前都会显示这是哪个主机的输出，但是看起来仍然很乱。

对于返回的内容，Fabric提供七种内容：标准输出、标准错误、警告消息、放弃消息、状态消息、执行信息、用户信息。对于这些内容，Fabric可设置输出哪些隐藏哪些。MCollective返回一个结果对象，在内容上也能很好满足输出要求。在输出格式上可以做很多自定义设置。

***

####其他比较####

<strong>语言差异</strong>
    
Fabric基于Python，也是python类库。MCollective基于Ruby。

<strong>难易程度</strong>
    
无论是部署还是使用，Fabric都远远比简单MCollective简单。MCollective更多是提供了一种框架，一种解决方案，需要很多自定义来选择。当然这也是说明了它具有很好的可扩展性。在部署上尤其是考虑到节点过多，网络复杂，延迟问题时，需要使用SubCollectives。这也增加了中间件及其他的部署复杂程度。

MCollective执行命令简单示例

<strong>代理程序</strong>

    module MCollective
        module Agent
            class Helloworld<RPC::Agent
                metadata :name        => "SimpleRPC Sample Agent",
                         :description => "Echo service for MCollective",
                         :author      => "R.I.Pienaar",
                         :license     => "GPLv2",
                         :version     => "1.1",
                         :url         => "http://projects.puppetlabs.com/projects/mcollective-plugins/wiki",
                         :timeout     => 60
                # Basic echo server
                action "echo" do
                    validate :msg, String
                    reply[:msg] = request[:msg]
                    reply[:status] = run(request[:msg], :cwd=>"/home", :stdout => :out, :stderr => :err)
                end
            end
        end
    end

<strong>客户端程序</strong>

    class MCollective::Application::Echo<MCollective::Application
       description "Reports on usage for a specific fact"

       option :message,
              :description    => "Message to send",
              :arguments      => ["-m", "--message MESSAGE"],
              :validation     => '^[\w[\s]\w]+$'

       def main
          mc = rpcclient("helloworld")

          printrpc mc.echo(:msg => configuration[:message], :options => options)

          printrpcstats
       end
    end

<strong>执行结果</strong>

    [root@master agent]# mco echo -m date -v
    Determining the amount of hosts matching filter for 2 seconds .... 2
     * [ ============================================================> ] 2 / 2


    puppet.example.com                      : OK
        {:status=>0, :msg=>"date", :out=>"Fri Jul 27 14:49:34 UTC 2012\n", :err=>""}

    master.example.com                      : OK
        {:status=>0, :msg=>"date", :out=>"Fri Jul 27 15:05:01 EDT 2012\n", :err=>""}



    ---- rpc stats ----
               Nodes: 2 / 2
         Pass / Fail: 2 / 0
          Start Time: Fri Jul 27 15:05:01 -0400 2012
      Discovery Time: 2004.03ms
          Agent Time: 122.94ms
          Total Time: 2126.97ms


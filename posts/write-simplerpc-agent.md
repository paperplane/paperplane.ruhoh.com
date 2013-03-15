---
title: Write SimpleRPC Agent
date: '2013-03-15'
description: 
categories: ['MCollective','OPS']
tags: ['Mcollective','SimpleRPC']
---
##编写SimpleRPC代理##

SimpleRPC能够工作是因为它假设了如何来写Agents。下面通过完整的HelloWorld代理来演示。首先需要明白客户端发送数据格式，明白数据接收规则。

****

####接收数据规则####

客户端发送请求如下格式：

    mc.echo(:msg => "Welcome to MCollective Simple RPC")

更复杂的例子：
    
    exim.setsender(:msgid => "1NOTVx-00028U-7G", :sender => "foo@bar.com")

这为成员变量：msgid和:sendid创建Hash，当然也可以直接使用string:

    exim.setsender("msgid" => "1NOTVx-00028U-7G", "senderid" => "foo@bar.com")

只要安全插件支持，数据类型可以保持不变，这也是默认做法。可以传递数组，哈希，哈希的哈希,openstructs但要保证传进来这些是key/value形式。

数据项不能使用:process_results，它对代理和客户端有特殊含义，这暗示代理客户端不会等待进程结果，一般用在不会选择收到回复。

***

####示例代理####

以下是一个Helloworld的代理，将输入的参数msg的值输出。


****
    module MCollective
        module Agent
            class Helloworld<RPC::Agent
                # Basic echo server
                action "echo" do
                    validate :msg, String
                    reply[:msg] = request[:msg]
                end
            end
        end
    end

这是一个能工作却不完全的代理。没有帮助没有Metadata.执行过程及结果

    [root@master agent]# mco rpc  --agent helloworld --action echo --argument msg="Welcome to MCollective Simple RPC"
    Determining the amount of hosts matching filter for 2 seconds .... 2
     * [ ============================================================> ] 2 / 2
    Finished processing 2 / 2 hosts in 65.70 ms

<strong>代理名称 AgentName</strong>

代理名称由类名继承而来，示例代码类名 MCollective::Agent::Helloworld那么代理名称就是helloworld

<strong>目标数据和初始化</strong>

SimpleRPC仍需要目标数据，没有就只能是默认设置。加了目标数据的代码如下：

    module MCollective
        module Agent
            class Helloworld<RPC::Agent
                metadata :name        => "SimpleRPC Sample Agent",
                         :description => "Echo service for MCollective",
                         :author      => "JunQi Lee",
                         :license     => "GPLv2",
                         :version     => "1.1",
                         :url         => "http://projects.puppetlabs.com/projects/mcollective-plugins/wiki",
                         :timeout     => 60
                # Basic echo server
                action "echo" do
                    validate :msg, String
                    reply[:msg] = request[:msg]
                end
            end
        end
    end

增加的代码设置了创建者信息，license和版本以及timeout。timeout是在杀掉该代理前允许代理运行的时间。如果设置过短，代理执行完之前就被终结。

<strong>写动作 Write Actions</strong>

动作（actions）是代理允许执行的单个任务：

    action "echo" do
        validate :msg, String
        
        reply[:msg] = request[:msg]
    end

这里创建了一个名为'echo'的动作，它没有任何参数

***

####激活代理####

过去仅仅需要将代理拷贝到机器上就能运行，因为所有代理无论依赖都被激活。

为了使部署简单并且支持代理选择特定运行平台，默认情况下代理能够被配置是否激活。

    plugin.helloworld.activate_agent = false

你可已在下面的文件中 /etc/mcollective/plugins.d/helloworld.cfg加入这句话（这个文件就是根据代理名在安装目录下的plugin.d目录下自己添加的）：

    activate_agent = false

这是启用和禁用代理最简单的方式。代理也可已在它的定义文件中声明何时被激活：

    module MCollective
        module Agent
            class Helloworld<RPC::Agent

                activate_when do
                    File.executable?("/usr/bin/puppet")
                end
            end
        end
    end

如果此块返回错误或者异常，此台机器上该代理不会被激活也不会被发现。每次代理加载时都会测试/usr/bin/puppet是否存在，只有存在该代理才会被激活。

***

####帮助和数据描述语言（DDL）####

代理ruby文件之外同时还有一个单独文件用于详细描述代理，之前例子的DDL文件如下所示：（注：此处文档语法多处错误，且文档多处单词错误）

    metadata :name        => "SimpleRPC Sample Agent",
             :description => "Echo service for MCollective",
             :author      => "R.I.Pienaar",
             :license     => "GPLv2",
             :version     => "1.1",
             :url         => "http://projects.puppetlabs.com/projects/mcollective-plugins/wiki",
             :timeout     => 60

    action "echo", :description => "Echos back any message it receives" do
        input :msg,
              :prompt      => "Service Name",
              :description => "The service to get the status for",
              :type        => :string,
              :validation  => '^[a-zA-Z\-_\d]+$',
              :optional    => false,
              :maxlength   => 30

        output :msg,
               :description => "The message we received",
               :display_as  => "Message"
    end

如上所示，DDL文件语法很简单就是一些标记值，帮助和其他重要的验证信息。DDL语言详细信息见数据描述语言DDL部分。

<strong>验证输入</strong>

如果按照规则用Hash结构接收发送的数据，然后就可以使用提供的验证器来确定接收的数据正是想要的数据。如果没有采用输入哈希结构，验证器不会起作用。今后基于DDL验证会自动执行，所以强烈建议使用哈希。

在示例的动作里验证：msg输入是String。这有些例子：

    1    validate :msg, /[a-zA-Z]+/
    2    validate :ipaddr, :ipv4address
    3    validate :ipaddr, :ipv6address
    4    validate :commmand, :shellsafe
    5    validate :mode, ["all", "packages"]

下面的列表展示所有支持的验证器：

    类型    说明    举例
    正则表达式  将输入与提供的正则表达式匹配    validate :msg, /[a-zA-Z]+/
    Type    验证输入是给定ruby数据类型  validate :msg, String
    IPv4    验证IPv4地址    validate :ipaddr, :ipv4address
    IPv6    验证IPv6地址    validate :ipaddr, :ipv6address
    system call safety  确保输入没有><半括号，管道符号之类  validate :command, :shellsafe
    Boolean 验证输入时true 或者false    validate :enable, :bool
    List of valid options   验证输入是所给列表中之一    validate :mode, [“all”, “packages”]

几点说明：

所有这些检查会引发InvalidRPCDate异常，Simple RPC框见会在适当时候捕获处理所以不用去catch。

可以提供自己的输入验证器，自定义验证类型。

另外如果想让某个String不被传入Shell，可以使用新版本ruby的shellescape来避免：

    safe = shellescape(request[:foo])

<strong>代理配置</strong>

可以在server.cfg中配置代理

    plugin.helloworld.setting = foo

上面的配置等同于：
    
    setting = config.pluginconf["helloworld.setting"] || ""

这会使任何未设置选项设置为“”


<strong>访问输入</strong>

输出可以根据repuest很容易获取，这会是之前向客户端输入的原始请求的Hash值。

request对象是MCollective::RPC::Request的实例，可以获取以下访问全权限：

属性    描述
time    消息发送时间
action  发送的要求代理执行的动作
data    数据的哈希值
sender  发送者的senderid
agent   往哪个代理发送

通过这种形式：request.data[:msg]或者request[:msg]

注：根据第一种方式会给你对所有正常哈希方法的访问权限，但是第二种只能获取include中的访问权限。


***

####执行Shell命令####

存在一个帮助函数能够事运行Shell命令变得很容易，也可以获取STDOUT,STDERR。推荐所有人使用这个函数来调用shell命令，因为它强制LC_ALL为C而且等待所有的子进程并且避免僵尸进程，可以给它设置唯一的工作路径和shell环境（这个简单地使用ruby提供的system可能无法实现）

最简单地用例就是执行一个命令并且向客户端返回输出：

    reply[:status] = run("echo 'hello world'", :stdout => :out, :stderr => :err)

这里可以设置基于命令输出的reply[:out], reply[:err]和 reply[:status]
    
也可以将输出附加到任意string：

    out = []
    err = ""
    status = run("echo 'hello world'", :stdout => out, :stderr => err)

例子中命令的STDOUT将被存储到变量out中并且不会被送回调用者。唯一需要注意的就是变量out和err有<<方法，可以将输出每一行应用到一个数组一个成员。在例子中out将会是一个拥有多行的数组而err就是一个多行的string.

    [root@master application]# mco rpc helloworld echo msg="uname" -v
    Determining the amount of hosts matching filter for 2 seconds .... 2

     * [ ============================================================> ] 2 / 2

    puppet.example.com                      : OK
        {:status=>0, :msg=>"uname", :out=>"Linux\n", :err=>""}

    master.example.com                      : OK
        {:msg=>"uname", :out=>"Linux\n", :err=>""}

    ---- helloworld#echo call stats ----
               Nodes: 2 / 2
         Pass / Fail: 2 / 0
          Start Time: Wed Jul 25 13:29:02 -0400 2012
      Discovery Time: 2002.91ms
          Agent Time: 119.73ms
          Total Time: 2122.64ms


默认情况下，任何尾随的换行符将被包含在输出和错误:
    
    reply[:status] = run("echo 'hello world'", :stdout => :out, :stderr => :err)
    reply[:stdout].chomp!
    reply[:stderr].chomp!

如果希望从目录/tmp处运行

    reply[:status] = run("echo 'hello world'", :stdout => :out, :stderr => :err, :cwd => "/tmp")

或者希望包含环境变量：
    
    reply[:status] = run("echo 'hello world'", :stdout => :out, :stderr => :err, :environment => {"FOO" => "BAR"})

返回状态是程序执行的返回码，如果程序完全失败比如文件不存在、资源不可用，返回码将是-1。

必须通过这些选项来设置CWD和环境变量。不要在代理中简单地调用chdir和ENV，这些在ruby多线程应用程序中不安全。


关于运行Shell命令部分，源代码主要是在actionrunner.rb中定义。ActionRunner类有如下属性:command, :agent, :action, :format, :stdout, :stderr, :request

其中run函数接收三个参数：command, request, format=:json，命令内容，请求内容，返回格式（默认json）.现在只有基于json序列化数据支持今后可能会增加基于key=val形式的数据。

根据源代码，整个类主要是通过把请求对象序列化为一个输入文件（临时文件）并且创建一个空的输出文件，然后创建外部命令读取输出文件。

任何标准输出将会在info级别计入日志，而标准错误将会在error级别记录日志。

执行命令过程中调用封转的外部命令MCollective::Shell。而shell又是封装的ruby systemu执行系统命令，systemu号称在获取标准输出和标准错误和处理子进程上是万能的。Shell有属性：:environment, :command, :status, :stdout, :stderr, :stdin, :cwd。程序输出经过精心封装，可根据返回码判断正确执行与否。

***

####结构化回复####

<strong>回复数据</strong>

回复数据在变量reply中，它是MCollective::RPC::Reply的实例。

    reply[:msg] = request[:msg]

<strong>回复状态</strong>

状态列表见写客户端部分。

    def rmmsg_action
       validate :msg, String
       validate :msg, /[a-zA-Z]+-[a-zA-Z]+-[a-zA-Z]+-[a-zA-Z]+/
       reply.fail "No such message #{request[:msg]}", 1 unless have_msg?(request[:msg])

       # check all the validation passed before doing any work
       return unless reply.statuscode == 0

       # now remove the message from the queue
    end

***

####外部脚本动作####

这部分可参考自定节点部分。动作可以使用其他支持JSON的外部语言实现。

    action "test" do
        implemented_by "script.py"
    end

此外这部分的授权、验证、审计等内容参考本篇其他相应部分。

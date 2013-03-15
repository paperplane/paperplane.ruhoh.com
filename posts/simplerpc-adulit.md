---
title: SimpleRPC Auditing
date: '2013-03-15'
tags: ['Mcollective','SimpleRPC']
categories: ['Mcollective','OPS']
---
##SimpleRPC审计##

作为SimpleRPC框架的一部分，审计系统用来将接收到的所有请求记录日志（在一个文件中）或者将这些日志发送到一个中心审计系统。审计数据同样可插件化而且可以根据自身需求来提供相应插件。

同授权一样，客户端在请求中会包括运行客户端库的进程的UID，而审计函数能够获取请求的访问权限。

***

####配置####

启用日志功能需要设置相应选项来启用和使用什么插件：

    rpcaudit = 1
    rpcauditprovider = Logfile

这将设置 MCollective::Audit::Logfile插件来记录事件日志。

客户端将会内嵌一个主叫ID，即执行客户端程序或者SSL证书的UNIX UID，这能够在请求对象中找到。

####Logfile 插件####

审计通过插件实现，将插件安装在正常插件目录mcollective/audit/下，下面是一个示例Logfile插件：

    module MCollective
        module RPC
            class Logfile<Audit
             require 'pp'

                def audit_request(request, connection)
                    logfile = Config.instance.pluginconf["rpcaudit.logfile"] || "/var/log/mcollective-audit.log"

                    now = Time.now
                    now_tz = tz = now.utc? ? "Z" : now.strftime("%z")
                    now_iso8601 = "%s.%06d%s" % [now.strftime("%Y-%m-%dT%H:%M:%S"), now.tv_usec, now_tz]

                    File.open(logfile, "w") do |f|
                        f.puts("#{now_iso8601}: reqid=#{request.uniqid}: reqtime=#{request.time} caller=#{request.caller}@#{request.sender} agent=#{request.agent} action=#{request.action} data=#{request.data.pretty_print_inspect}")
                    end
                end
            end
        end
    end

正如你看到的，只需要提供函数audit_request，可以获得MCollective::RPC::Request 对象的请求形式和建立中间件连接。

此外需要做如下配置（在server.cfg）中：

    plugin.rpcaudit.logfile = /var/log/mcollective-audit.log

最后结果类似下面：
    2010-12-28T17:09:03.889113+0000: reqid=319719cc475f57fda3f734136a31e19b: reqtime=1293556143 caller=cert=nagios@monitor1 agent=nrpe action=runcommand data={:process_results=>true, :command=>"check_mailq"}

社区中还提供有其他插件-中心化 RPC审计日志，这个插件将所有SimpleRPC审计事件发送到一个中心点来记录日志。


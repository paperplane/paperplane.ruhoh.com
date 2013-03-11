---
title: ZABBIX API简介及使用
date: '2013-01-19'
description:
categories: ['Zabbix','OPS']
tags: Zabbix
---

###API简介###

Zabbix API开始扮演着越来越重要的角色，尤其是在集成第三方软件和自动化日常任务时。很难想象管理数千台服务器而没有自动化是多么的困难。Zabbix API为批量操作、第三方软件集成以及其他作用提供可编程接口。

Zabbix API是在1.8版本中开始引进并且已经被广泛应用。所有的Zabbix移动客户端都是基于API，甚至原生的WEB前端部分也是建立在它之上。Zabbix API 中间件使得架构更加模块化也避免直接对数据库进行操作。它允许你通过JSON RPC协议来创建、更新和获取Zabbix对象并且做任何你喜欢的操作【当然前提是你拥有认证账户】。

Zabbix API提供两项主要功能：

+ 远程管理Zabbix配置

+ 远程检索配置和历史数据

<strong>使用JSON</strong>

API 采用JSON-RPC实现。这意味着调用任何函数，都需要发送POST请求，输入输出数据都是以JSON格式。大致工作流如下：

+ 准备JSON对象，它描述了你想要做什么（创建主机，获取图像，更新监控项等）。

+ 采用POST方法向http://example.com/zabbix/api_jsonrpc.php发送此JSON对象. http://example.com/zabbix/是Zabbix前端地址。api_jsonrpc.php是调用API的PHP脚本。可在安装可视化前端的目录下找到。

+ 获取JSON格式响应。

+ 注：请求除了必须是POST方法之外，HTTP Header Content-Type必须为【application/jsonrequest，application/json-rpc，application/json】其中之一。

可以采用脚本或者任何"手动"支持JSON RPC的工具来使用API。而首先需要了解的就是如何验证和如何使用验证ID来获取想要的信息。后面的演示会以Python脚本和基于Curl的例子来呈现API的基本使用。

<strong>基本请求格式</strong>

Zabbix API 简化的JSON请求如下：
    
    {
        "jsonrpc": "2.0",
        "method": "method.name", 
        "params": {
            "param_1_name": "param_1_value",
            "param_2_name": "param_2_value" 
        },
        "id": 1,
        "auth": "159121b60d19a9b4b55d49e30cf12b81",
    }

下面一行一行来看：

+ "jsonrpc": "2.0"-这是标准的JSON RPC参数以标示协议版本。所有的请求都会保持不变。 

+ "method": "method.name"-这个参数定义了真实执行的操作。例如：host.create、item.update等等

+ "params"-这里通过传递JSON对象来作为特定方法的参数。如果你希望创建监控项，"name"和"key_"参数是需要的，每个方法需要的参数在Zabbix API文档中都有描述。

+ "id": 1-这个字段用于绑定JSON请求和响应。响应会跟请求有相同的"id"。在一次性发送多个请求时很有用，这些也不需要唯一或者连续

+ "auth": "159121b60d19a9b4b55d49e30cf12b81"-这是一个认证令牌【authentication token】用以鉴别用户、访问API。这也是使用API进行相关操作的前提-获取认证ID。
* * *

###API 使用###

+ <strong>环境准备</strong>

Zabbix API是基于JSON-RPC 2.0规格，具体实现可以选择任何你喜欢的编程语言或者手动方式。这里我们采用的Python和基于Curl的方式来做示例。Python 2.7版本已经支持JSON，所以不再需要其他模块组件。当然可以采用Perl、Ruby、PHP之类的语言，使用前先确保相应JSON模块的安装。

+ <strong>身份验证</strong>

任何Zabbix API客户端在真正工作之前都需要验证它自身。在这里是采用User.login方法。这个方法接受一个用户名和密码作为参数并返回验证ID，一个安全哈希串用于持续的API调用（在使用User.logout之前该验证ID均有效）。具体Python代码auth.py如下：

    #!/usr/bin/env python2.7
    #coding=utf-8

    import json
    import urllib2

    # based url and required header
    url = "http://monitor.example.com/api_jsonrpc.php"
    header = {"Content-Type": "application/json"}

    # auth user and password
    data = json.dumps(
    {
        "jsonrpc": "2.0",
        "method": "user.login",
        "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 0
    })

    # create request object
    request = urllib2.Request(url,data)
    for key in header:
        request.add_header(key,header[key])

    # auth and get authid
    try:
        result = urllib2.urlopen(request)
    except URLError as e:
        print "Auth Failed, Please Check Your Name And Password:",e.code
    else:
        response = json.loads(result.read())
        result.close()
        print "Auth Successful. The Auth ID Is:",response['result']

这里需要确保URL中的用户名和密码匹配。下面是运行结果：

![picture1]({{urls.media}}/zabbix-1.png)

可以看到，auth.py成功连接并认证。现在有了验证ID，它能够在新的API调用中被重用。 

下面再来看基于CURL的方式来进行验证是如何实现的：

    curl -i -X POST -H 'Content-Type:application/json' -d'{"jsonrpc": "2.0","method":"user.authenticate","params":{"user":"Admin","password":"zabbix"},"auth": null,"id":0}' http://monitor.example.com/api_jsonrpc.php

![picture2]({{urls.media}}/zabbix-2.png)

+ <strong>一般操作</strong>
     
这里举例说明如何获取监控主机列表【host list】。这段脚本需要采用auth.py中获取的验证ID并执行host.get方法来获取主机列表。来看具体代码get_host.py:

    #!/usr/bin/env python2.7
    #coding=utf-8

    import json
    import urllib2

    # based url and required header
    url = "http://monitor.example.com/api_jsonrpc.php"
    header = {"Content-Type": "application/json"}

    # request json
    data = json.dumps(
    {
        "jsonrpc":"2.0",
        "method":"host.get",
        "params":{
            "output":["hostid","name"],
            "filter":{"host":""}
        },
        "auth":"2ee379e516f386ca4c24da7fd9fd5bb4", # the auth id is what auth script returns, remeber it is string
        "id":1,
    })

    # create request object
    request = urllib2.Request(url,data)
    for key in header:
        request.add_header(key,header[key])

    # get host list
    try:
        result = urllib2.urlopen(request)
    except URLError as e:
        if hasattr(e, 'reason'):
            print 'We failed to reach a server.'
            print 'Reason: ', e.reason
        elif hasattr(e, 'code'):
            print 'The server could not fulfill the request.'
            print 'Error code: ', e.code
    else:
        response = json.loads(result.read())
        result.close()

        print "Number Of Hosts: ", len(response['result'])

        for host in response['result']:
            print "Host ID:",host['hostid'],"Host Name:",host['name']

部分结果列表：

![picture3]({{urls.media}}/zabbix-3.png)

对比基于CURL的访问方式：
    curl -i -X POST -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","method":"host.get","params":{"output":["hostid","name"],"filter":{"host":""}},"auth":"ecc543db662930c122b5fbedee60cc63","id":1}' http://monitor.example.com/api_jsonrpc.php    

![picture4]({{urls.media}}/zabbix-4.png)

结果太多，未予显示。比较来看，采用脚本可以有更多的灵活性，而基于CURL的方式，对结果的处理不是很方便。原理则都是相通的。

除了这些获取信息以外，采用API调用同样可以进行创建操作，更新操作和删除操作等等。这也很容易让我们联想起数据库操作，当然比较这些采用API调用获取结果的方式，也不能忘掉这种最直接而危险的方式。在开始讨论中已经提到，Zabbix现在自带的前端实现部分是采用数据库操作，部分是基于API调用。在API还不是很成熟的现在，具体采用哪种方式，需要根据业务需求再来确定。
     
+ <strong>数据流程</strong>

下面的流程图代表了Zabbix API 工作的典型工作流。验证（方法user.login）是获取验证ID的强制步骤。这个ID又允许我们调用API提供的任何权限允许的方法来进行操作。在之前的例子中没有提到user.logout方法，这也是一次验证ID能够重复使用的原因所在。使用user.logout方法后将会使验证ID失效，后面的操作将不能再使用此ID。

![picture5]({{urls.media}}/zabbix-5.png)

* * *

###参考链接###

+ http://doc.bonfire-project.eu/R2/monitoring/monitoring_zabbix_API.html 
+ https://www.zabbix.com/documentation/1.8/api/getting_started
+ http://blog.zabbix.com/getting-started-with-zabbix-api/1381/



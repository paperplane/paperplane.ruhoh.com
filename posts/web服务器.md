---
title: web server
date: '2013-03-10'
description:
categories: Notes
tags: ['HTTP','Note']
---

WEB服务器部分主要包括三部分内容：

    常见WEB服务器类型
    编写简单WEB服务器
    WEB服务器工作步骤

本部分介绍的内容都是概括性、总结性的内容，后面的内容会对本部分提到的很多东西进行深入挖掘。
![图片]({{urls.media}}/http/webserver.jpeg)

***

###WEB服务器市场概况###

主要来看下Netcraft的最新市场调研结果

![图片]({{urls.media}}/http/webserver1.png)

![图片]({{urls.media}}/http/webserver2.png)

![图片]({{urls.media}}/http/webserver3.png)

不难看到，这次调研中已经有超过6亿的网站有HTTP响应。就最近几个季度数据整体来看，Apache，微软等的市场份额都有一定的下降，唯有Nginx，一直涨幅明显。在大型商业站点，Apache仍是一枝独秀，而Nginx则直追微软，可谓是WEB服务器中的黑马，战斗机。

详细信息： [netcraft](http://news.netcraft.com/archives/2013/03/01/march-2013-web-server-survey.html)

***

###Nginx VS Apache###
    
#####Nginx和Apache简介#####
    
Nginx是俄罗斯人编写的十分轻量级的HTTP服务器,Nginx，它的发音为“engine X”， 是一个高性能的HTTP和反向代理服务器，同时也是一个IMAP/POP3/SMTP 代理服务器．Nginx是由俄罗斯人 Igor Sysoev为俄罗斯访问量第二的 Rambler.ru站点开发的，它已经在该站点运行超过两年半了。Igor Sysoev在建立的项目时,使用基于BSD许可。
    
Apache是世界排名第一的web服务器.1995年4月, 最早的apache(0.6.2版)由apache group公布发行. apache group 是一个完全通过internet进行运作的非盈利机构, 由它来决定apache web服务器的标准发行版中应该包含哪些内容. 准许任何人修改隐错, 提供新的特征和将它移植到新的平台上, 以及其它的工作. 当新的代码被提交给apache group时, 该团体审核它的具体内容, 进行测试, 如果认为满意, 该代码就会被集成到apache的主要发行版中. 
    
Nginx作为"年轻一代",在许多方面都比竞争对手更高效。

1 速度：利用异步套接字，接到请求时，不会派生出与请求一样多的子进程。考虑减轻CPU负载和内存消耗，每个核一个进程足以处理数千个连接。
    
2 易用：与Apache等相比，配置文件的读取和调整都十分容易，只需几行就可以建立一个完整的虚拟主机。
    
3 模块性：自带强大插件系统--"模块"，原始发布归档文件与第三方包含丰富模块可供下载。
    
Nginx集速度、效率和能力于一体，是迄今为止，相对于Apache来说最好的选择。
    
#####Nginx基本入门指导----从安装到配置#####

    /usr/sbin/groupadd www -g 48
    /usr/sbin/useradd -u 48 -g www -s /sbin/nologin www

    wget http://nginx.org/download/nginx-1.1.15.tar.gz
    wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-7.8.tar.bz2

    cd /usr/local/
    tar jxvf /home/lijunqi/pcre-7.8.tar.bz2 
    cd pcre-7.8/
    ./configure 
    make && make install

    cd ..
    tar zxvf /home/lijunqi/nginx-1.1.15.tar.gz
    cd nginx-1.1.15/
    ./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module
    make && make install
    cd ..
    /usr/local/nginx/sbin/nginx

***

###编写简单WEB服务器###

#####Python实现#####

    最简单的方式：

    python -m SimpleHTTServer

    简单服务器：

    import socket

    # Create TCP/IP Socket
    sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

    # Bind the Socket To the Port
    server_address = ('localhost', 2048)
    sock.bind(server_address)

    # Listen for Incoming Connetions
    sock.listen(5)

    while True:
        # Wait for Connection
        connection, client_address = sock.accept()

        try:
            # receive data and retransmit it
            while True:
                data = connection.recv(16)
                if data:
                    connection.sendall(data)
                else:
                    break
        finally:
            # Clean up the connection
            connection.close()

***

###WEB服务器工作流程###

#####基本流程及处理问题#####

这些步骤很多内容都会在后面分别讲到，这里只是简单概括性总结。

建立连接--接受客户端连接，或者不希望与该客户端建立连接，就将其关闭

接收请求--从网络中读取一条HTTP请求报文

处理请求--对请求报文进行解释，并采取行动

访问资源--访问报文中指定的资源

构建响应--创建带有正确首部的HTTP响应报文

发送响应--将响应回送给客户端

记录日志--记录事务处理过程，将与已完成事务相关的内容记录在日志文件中
    

***

+ 参考资料
+ [HTTP权威指南](http://book.douban.com/subject/10746113/)
+ [Netcraft](http://news.netcraft.com/)

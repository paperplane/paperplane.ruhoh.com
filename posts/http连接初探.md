---
title:
date: '2013-01-03'
description:
categories:
---

###概述###

本部分主要讨论HTTP连接管理及其相关话题。在此之前，需要先对TCP连接及其性能相关问题进行研究。简单原因如下：

+ HTTP连接实际上就是TCP连接及其使用规则
+ 就HTTP时延而言，TCP网络时延是主要部分

* * *

###TCP协议相关###

+ 浏览器与服务器交互过程图解
![broswer-server]({{urls.media}}/broswer-server.png)

过程大致如下：

DNS解析 -> TCP连接 -> HTTP请求 -> HTTP处理 -> HTTP返回 -> 连接关闭

+ HTTP协议结构与TCPIP数据包
![proctol]({{urls.media}}/proctol.png)

+ TCP套接字接口通信编程实例

Python接口
    
操作系统提供了操纵其TCP连接的工具-TCP编程接口。即常说的Socket编程。这里我们主要来看下Python语言提供的相关API，并使用这些API编写简单的客户端服务器程序并进行相关通信。套接字API最初为Unix操作系统开发，但现在几乎所有的操作系统和编程语言都有其变体。Python库通过Socket模块支持套接字。Socket模块还提供了一个工厂函数，也成为socket,调用该函数生成套接字对象，然后再调用该对象的方法进行网络层操作。因为并不是系统介绍Python Socket编程，以下只列举相关可能会用到的函数和方法，更多请参考[Python文档]( http://docs.python.org/2/library/socket.html)。

Socket模块常用函数表【部分函数】

|函数原型|函数描述|
|-------------|-------------|
|getdefaulttimeout()|返回浮点型值，该值为新创建的套接字对象当前默认的超时时间【按秒计算】，若新创建为负值返回None|
|getfqdn(host='')|返回给定host的完整域名字符串。在host为''时，返回本地主机的完整域名字符串|
|gethostbyaddr(ipaddr)|返回元组(hostname,alias_list,ipaddr_list),即该IP地址主机的主名称，别名列表，点分十进制IP地址列表|
|gethostbyname_ex(hostname)|与gethostbyaddr返回相同结果，但是hostname字符串参数可以是点分十进制IP地址或者DNS名称|
|setdefaulttimeout(t)|设置浮点值t为新创建套接字对象的默认超时时间，以秒为单位。若t为None,则无超时时间设置|
|socket (socket_family, socket_type, protocol=0)|创建并返回具有给定famliy和type的套接字对象。socket_family: 这是AF_UNIX或AF_INET，传输机制所使用的协议族.socket_type: 这是SOCK_STREAM，或为SOCK_DGRAM，表示UPD或者TCP连接.protocol: 这通常被不用关心，默认为0.|

Socket对象常用方法表【常见方法】
|方法名称|方法描述|
|-------------|-------------|
|s.bind()|绑定地址(主机，端口)到套接字|
|s.listen()|开始监听|
|s.accept()|被动接受 tcp客户端连接(阻塞式)，等待连接的到来|
|s.connet()|主动初始化tcp服务器连接|
|s.connet_ex()|connet扩展版本，出错时返回错误代码，不抛出异常|
|s.recv()|接受TCP数据|
|s.send()|发送TCP数据|
|s.recvfrom()|接受UDP数据|
|s.sendto()|发送UDP数据|  
|s.close()|关闭套接字|


服务器实现

    import socket
    import sys 

    # Create TCP/IP Socket
    sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

    # Bind the Socket To the Port 
    server_address = ('localhost', 2048)
    print >> sys.stderr, 'starting up on %s port %d' % server_address
    sock.bind(server_address)

    # Listen for Incoming Connetions
    sock.listen(5)

    while True:
        # Wait for Connection
        print >> sys.stderr, 'waiting for a connection'
        connection, client_address = sock.accept()

        try:
            print >>sys.stderr, 'connection from', client_address

            # receive data and retransmit it
            while True:
                data = connection.recv(16)
                print >>sys.stderr, 'received "%s" ' % data
                if data:
                    print >>sys.stderr, 'sending back data to the client'
                    connection.sendall(data)
                else:
                    print >> sys.stderr, 'no data from', client_address
                    break
        finally:
            # Clean up the connection
            connection.close()
客户端实现
    
    import socket

    # Create TCP/IP Socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Connect the Socket to the Port where server is listenting
    server_address = ('localhost', 2048)
    sock.connect(server_address)

    try:
        # send data
        message = 'This is a Message, to be sent for a test. Every time sent 16 byte until all messages has been sent all.'
        sock.sendall(message)

        # test transmit 
        num_received = 0 
        num_excepted = len(message)
        while num_received < num_excepted:
            data = sock.recv(16)
            num_received += len(data)
    
    finally:
        sock.close()

+ HTTP事务时延与TCP网络时延

HTTP事务时延构成
![http-delay]({{urls.media}}/http-delay.png)

由上图看网络时延的构成：

解析时延
    DNS解析与DNS缓存

连接时延
    TCP连接的建立

传输时延
    HTTP请求发送
    HTTP响应返回

处理时延
    HTTP报文处理

与建立TCP连接，以及传输请求和响应报文的时间相比，事务处理时间可能是最短的。除非客户端或服务器超载，或正在处理复杂的动态资源，否则HTTP时延主要就是TCP网络时延。
     
TCP网络时延的大小取决于硬件速度、网络和服务器负载，请求和响应报文的尺寸，以及客户端和服务器之间的距离。TCP技术的复杂性也会对时延产生较大的影响。

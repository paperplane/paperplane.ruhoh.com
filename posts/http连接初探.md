---
title: HTTP连接初探
date: '2013-01-03'
description: 
categories: Notes
tags: ['HTTP','Note']
---

###概述###

本部分主要讨论HTTP连接管理及其相关话题。在此之前，需要先对TCP连接及其性能相关问题进行研究。简单原因如下：

+ **HTTP连接实际上就是TCP连接及其使用规则**
+ **就HTTP时延而言，TCP网络时延是主要部分**

此外，由于HTTP连接内容较多，本篇文章篇幅有限，下一篇文章将继续深入研究关于持久化连接和管道化连接内容。

* * *

###TCP协议相关###

+ 浏览器与服务器交互过程图解
![broswer-server]({{urls.media}}/broswer-server.png)

过程大致如下：

**DNS解析 -> TCP连接 -> HTTP请求 -> HTTP处理 -> HTTP返回 -> 连接关闭**

+ HTTP协议结构与TCPIP数据包
![proctol]({{urls.media}}/proctol.png)

+ TCP套接字接口通信编程实例

**Python接口**
    
操作系统提供了操纵其TCP连接的工具-TCP编程接口。即常说的Socket编程。这里我们主要来看下Python语言提供的相关API，并使用这些API编写简单的客户端服务器程序并进行相关通信。套接字API最初为Unix操作系统开发，但现在几乎所有的操作系统和编程语言都有其变体。Python库通过Socket模块支持套接字。Socket模块还提供了一个工厂函数，也成为socket,调用该函数生成套接字对象，然后再调用该对象的方法进行网络层操作。因为并不是系统介绍Python Socket编程，以下只列举相关可能会用到的函数和方法，更多请参考[Python文档]( http://docs.python.org/2/library/socket.html)。

Socket模块常用函数表【部分函数】
<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>函数原型</td>
            <td>函数描述</td>
        </tr>
        <tr>
            <td>getdefaulttimeout()</td>
            <td>返回浮点型值，该值为新创建的套接字对象当前默认的超时时间【按秒计算】，若新创建为负值返回None</td>
        </tr>
        <tr>
            <td>getfqdn(host='')</td>
            <td>返回给定host的完整域名字符串。在host为''时，返回本地主机的完整域名字符串</td>
        </tr>
        <tr>
            <td>gethostbyaddr(ipaddr)</td>
            <td>返回元组(hostname,alias_list,ipaddr_list),即该IP地址主机的主名称，别名列表，点分十进制IP地址列表</td>
        </tr>
        <tr>
            <td>gethostbyname_ex(hostname)</td>
            <td>与gethostbyaddr返回相同结果，但是hostname字符串参数可以是点分十进制IP地址或者DNS名称</td>
        </tr>
        <tr>
            <td>setdefaulttimeout(t)</td>
            <td>设置浮点值t为新创建套接字对象的默认超时时间，以秒为单位。若t为None,则无超时时间设置</td>
        </tr>
        <tr>
            <td>socket (socket_family, socket_type, protocol=0)</td>
            <td>创建并返回具有给定famliy和type的套接字对象。socket_family: 这是AF_UNIX或AF_INET，传输机制所使用的协议族.socket_type: 这是SOCK_STREAM，或为SOCK_DGRAM，表示UPD或者TCP连接.protocol: 这通常被不用关心，默认为0.</td>
        </tr>
    </tbody>
</table>

Socket对象常用方法表【常见方法】
<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr><td>方法名称</td><td>方法描述</td></tr>
        <tr><td>s.bind()</td><td>绑定地址(主机，端口)到套接字</td></tr>
        <tr><td>s.listen()</td><td>开始监听</td></tr>
        <tr><td>s.accept()</td><td>被动接受 tcp客户端连接(阻塞式)，等待连接的到来</td></tr>
        <tr><td>s.connet()</td><td>主动初始化tcp服务器连接</td></tr>
        <tr><td>s.connet_ex()</td><td>connet扩展版本，出错时返回错误代码，不抛出异常</td></tr>
        <tr><td>s.recv()</td><td>接受TCP数据</td></tr>
        <tr><td>s.send()</td><td>发送TCP数据</td></tr>
        <tr><td>s.recvfrom()</td><td>接受UDP数据</td></tr>
        <tr><td>s.sendto()</td><td>发送UDP数据</td></tr>
        <tr><td>s.close()</td><td>关闭套接字</td></tr>
    </tbody>
</table>

**服务器实现**

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

**客户端实现**
    
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

**HTTP事务时延构成**
![http-delay]({{urls.media}}/http-delay.png)

由上图看网络时延的构成：

    解析时延    DNS解析与DNS缓存
    连接时延    TCP连接的建立
    传输时延    HTTP请求发送    HTTP响应返回
    处理时延    HTTP报文处理

与建立TCP连接，以及传输请求和响应报文的时间相比，事务处理时间可能是最短的。除非客户端或服务器超载，或正在处理复杂的动态资源，否则HTTP时延主要就是TCP网络时延。

TCP网络时延的大小取决于硬件速度、网络和服务器负载，请求和响应报文的尺寸，以及客户端和服务器之间的距离。TCP技术的复杂性也会对时延产生较大的影响。

**TCP网络时延分析**

1 TCP连接的握手时延
    
    + 建立连接之前的HTTP三次握手： SYN/SYN+ACK/ACK
    + 通常HTTP事务不会交换太多数据，SYN/SYN+ACK会产生一个可测量的时延
    + TCP连接的ACK分组都足够大，可以承载整个HTTP请求报文，而响应报文很多都可以放入一个IP分组
    + 最后的结果就是，小的HTTP事务会在TCP连接上花费50%，或者更多的时间
    + 改进：重用连接                  

2 TCP慢启动拥塞控制
    
    + 慢启动，用于防止因特网的突然过载和拥塞
    + 由于存在拥塞控制，新建连接传输速度会比已经交换过一定量数据的、'已调谐'的连接慢，所以就希望能够重用这些现存连接
    + 改进：持久连接

3 用于捎带确认的TCP延迟确认
    
    + 因特网无法确保可靠的分组传输，TCP协议实现自身的确认机制来保值数据的成功传输
    + 由于确认报文较小，TCP允许将返回的确认信息与同向的输出数据分组一起进行'捎带'，而为了增加捎带概率采用延迟确认算法
    + HTTP的双峰特征-请求应答行为降低了捎带信息的可能性，通常，延迟算法会导致一定的时延
    
4 数据聚集的Nagle算法

    + TCP发送大量包含数据的分组，会严重影响网络性能。Nagle算法试图在发送分组之前，绑定大量TCP数据，鼓励发送全尺寸的段，将数据缓存直至其他分组都被确认或者缓存中已足够全尺寸的段才会发送
    + 引入问题
        1 小的HTTP报文可能无法填满一个分组，可能会因为等待不会到来的数据产生时延
        2 与延迟确认算法交互存在问题。Nagle算法阻止数据发送，直到有确认分组抵达，但确认分组自身会被延迟确认算法延迟，因为它在等待捎带它的数据包

5 TIME_WAIT累积与端口耗尽

    + TIME_WAIT的作用：允许老的重复分组在网络中消失，防止最后ACK的丢失可靠地实现TCP全双工通信的终止
    + TCP连接四要素<源IP，源端口，目的IP，目的端口>，在一个客户端和一台服务器的情况下，其中三个都是固定的，只有源端口可以改变，客户端每次连接都会获得新的源端口，以实现连接唯一性
    + 由于源端口的数量有限，而且在2MSL时间内连接无法重用，服务器连接率就会受限
* * *

###HTTP连接管理###

+ 连接管理

1 Connection首部

    三种不同类型的标签
        HTTP首部字段名，列出了只与此连接相关的首部，而不会传播到其他连接中去。所以在此报文转发出去之前，这些首部必须被删除
        任意值标签，用于描述此连接的非标准选项
        值close,说明此操作完成之后需关闭这条持久连接
    connection首部可以防止无意中对本地首部的转发，因此将逐级跳首部名放入connection首部被称为对首部的保护
    HTTP应用程序收到connection首部的报文时，会进行解析所有选项并对其进行应用。然后在转发下一跳之前会删除connection首部和connection中列出的首部

2 连接关闭管理

    正常关闭连接：完全关闭与半关闭
    合理的关闭顺序：正常关闭的应用程序首先关闭输出信道，然后等待连接另一端的对等实体关闭它的输出信道。当两端都告诉对发不会再发送任何数据之后，连接就会完全被关闭
    半关闭引起的连接被对端重置错误
    任意解除连接与随时准备重新建立连接传输未确认分组，尤其是在持久连接和管道连接
    连接关闭时进行数据结尾操作会遇到的Content-Length不准确的处理方式

+ 连接优化

<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>连接名称</td>
            <td>描述</td>
            <td>优化思路</td>
            <td>优化方式</td>
            <td>优点</td>
            <td>缺点</td>
            <td>限制</td>
        </tr>
        <tr>
            <td>并行连接</td>
            <td>通过多条TCP连接发起并行的HTTP请求</td>
            <td>将连接时延与传输时延并行，从而减少时延</td>
            <td>客户端打开多条连接，并行执行多个HTTP事务</td>
            <td>可能会提高页面加载速度 感觉也会更快些</td>
            <td>每个事务建立新连接，会耗费时间和带宽 慢启动特性存在，新连接性能有所降低</td>
            <td>可打开的并行连接数量实际是有限的</td>
        </tr>
        <tr>
            <td>持久连接</td>
            <td>重用TCP连接，以消除连接及关闭时延</td>
            <td>消除连接时延</td>
            <td>HTTP/1.0 + Keep-alive和HTTP/1.1 Persistent </td>
            <td>减少cpu和内存占用(因为同一时间,启用更少的连接) 减轻网络堵塞(建立更少的连接) 减轻后续请求的延迟(因为避免建立新连接而减频繁的握手) 不需要牺牲当前的tcp连接, 就能够报告错误</td>
            <td>keep-alive的哑代理【忙中继】问题 对于单个文件被不断请求的服务，Keep-Alive可能会极大的影响性能，因为它在文件被请求之后还保持了不必要的连接很长时间</td>
            <td>每个持久连接适用于一跳传输 一个单用户客户端对于任何一台服务器或者代理服务器都可以维护不多于两个的连接数     在由当前由n台服务器组成的网络中, 任意一台代理服务器对另外的服务器或者代理服务器应该维护2*n个连接</td>
        </tr>
        <tr>
            <td>管道化连接</td>
            <td>通过共享的TCP连接发起并发的HTTP请求</td>
            <td>消除传输时延</td>
            <td>多个http请求不需要等待相应的应答就能够写进同一个socket</td>
            <td>http管道化向网络上发送更少的tcp数据包,以便减轻网络负载</td>
            <td>HTTP客户端必须做好连接会在任意时刻关闭，准备好重发所有未完成管道化请求</td>
            <td>只有幂等的请求才能管道化 按照与请求相同的顺序回送HTTP响应 HTTP客户端必须确认连接是持久的，才能使用管道 新建立的连接无法判断源服务器是否支持HTTP/1.1不能进行管道化</td>
        </tr>
    </tbody>
</table>

+ 相关概念
     
**非幂等请求**
     
假如在不考虑诸如错误或者过期等问题的情况下，若干次请求的副作用与单次请求相同或者根本没有副作用，那么这些请求方法就能够被视作“幂等”的。GET，HEAD，PUT和DELETE方法都有这样的幂等属性，同样由于根据协议，OPTIONS，TRACE都不应有副作用，因此也理所当然也是幂等的。
     
假如某个由若干个请求做成的请求串行产生的结果在重复执行这个请求串行或者其中任何一个或多个请求后仍没有发生变化，则这个请求串行便是“幂等”的。但是，可能出现若干个请求做成的请求串行是“非幂等”的，即使这个请求串行中所有执行的请求方法都是幂等的。例如，这个请求串行的结果依赖于某个会在下次执行这个串行的过程中被修改的变量。转自[维基百科](http://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE#.E5.B9.82.E7.AD.89.E6.96.B9.E6.B3.95)
          
**站点局部性**

初始化了（建立连接）对某服务器的HTTP请求的应用程序很可能会在不久的将来对那台服务器发起更多请求。

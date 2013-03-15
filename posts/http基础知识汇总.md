---
title: Introduction To HTTP
date: '2012-11-23'
description:
categories: Notes
tags: ['HTTP','Note']
---

###HTTP简介###

+ HTTP是万维网背后的协议，是每个成功WEB事务的幕后推手。每次对WEB文档或者图像的请求、每次对超链接的点击、每次对表格的提交等这些背后都是HTTP协议。
+ HTTP协议实现互联网中信息的分发与交互，它为计算机之间相互通信提供了标准方式。HTTP协议指定客户端怎样请求数据以及服务器如何响应这些请求。
+ HTTP是一个属于应用层的面向对象的协议，适用于分布式多媒体信息系统。HTTP协议是因特网的多媒体信使，并且使用可靠地数据传输协议，确保数据在传输过程中不会被损坏或产生混乱。
+ HTTP协议历经了HTTP/0.9、HTTP/1.0、HTTP/1.0+、HTTP/1.1的版本变化，现在HTTP-NG原型建议已经提出。1.0和1.1是广泛使用的两个版本。主要围绕特性调整、结构设计、性能优化等方面改进。

* * *
###HTTP特点###
+ <strong>客户端/服务器模式</strong>：最基本的通信模式。
+ <strong>简单迅速灵活</strong>：客户端请求时，只需指定请求方法和路径比较简单;HTTP服务器的程序规模小，通信迅速;HTTP允许传输任意类型数据对象，比较灵活。
+ <strong>【非】持续连接</strong>：既支持持续连接（1.1版本默认），也支持非持续连接（1.0版本默认）。
+ <strong>无状态</strong>：无状态即对事务的处理没有记忆能力。缺少状态意味着后续处理相同的信息，必须重传。

注：最简单的HTTP服务器实现估计是：python -m SimpleHTTPServer。

* * *

###HTTP举例###
![图片]({{urls.media}}/HTTP.png)

上述过程简单理解为一个人给另一个人打电话询问某个信息

询问者和被询问者是<strong>客户端</strong>和<strong>服务器</strong>，客户端向服务器请求信息，服务器响应客户端请求。

电话则相当于<strong>WEB浏览器</strong>，是客户端请求的一种媒介。

询问的过程就是<strong>HTTP事务</strong>（假设被询问是有问必答），包括请求过程和响应过程。

对话的内容则是<strong>HTTP报文</strong>（双方定义好对话的法则），请求和相应都有相应规定。

询问的信息便是<strong>WEB资源</strong>，这里是关于名为/的文档，资源既有类型也有相应名称。

再使用tcpdump查看报文内容，看看具体的电话内容：
![tcpdump抓包]({{urls.media}}/tcpdump_http.png)

* * *

###基本术语###
+ WEB结构组件/应用程序
    
    1 基本组件----客户端和服务器
    
    HTTP客户端和HTTP服务器构成了万维网的基本组件，WEB浏览器是最常见的客户端，客户端与服务器请求、响应的"对话方式"。
    
    2 常用组件
    
    在因特网中，要与很多WEB应用程序进行交互（不仅仅是WEB浏览器和WEB服务器），也就是说HTTP事务并不完全是像开始描述的仅仅是客户端服务器交互，多个WEB应用程序参与进来就会有不同的交互模式，但基本模式是这种客户端服务器模式。这里简单地做一些概括性汇总，详细比较分析后面有专门部分介绍。
<table class="table table-bordered table-striped table-condensed">
    <tr>
        <td>名称</td>
        <td>作用</td>
        <td>举例</td>
    </tr>
    <tr>
        <td>代理</td>
        <td>安全、应用集成以及性能优化重要组件，常见有透明代理、反向代理、正向代理</td>
        <td>客户端和服务器之间转发流量</td>
    </tr>
    <tr>
        <td>缓存</td>
        <td>将源站资源备份以备下次相同请求直接使用副本</td>
        <td>保存常用文档本地副本以提高性能</td>
    </tr>
    <tr>
        <td>网关</td>
        <td>用于将HTTP流量转化成其他协议，就像自己是源端服务器一样接受请求，对客户端透明</td>
        <td>HTTP/FTP网关</td>
    </tr>
    <tr>
        <td>隧道</td>
        <td>建立起来之后，就会在两条连接之间对原始数据进行盲转发，可转发非HTTP数据，且不会窥探数据</td>
        <td>HTTP连接承载加密的安全套接字层</td>
    </tr>
    <tr>
        <td>Agent代理</td>
        <td>代表用户发起HTTP请求。所有发布WEB请求的应用程序都是HTTP Agent 代理</td>
        <td>网络爬虫、浏览器等</td>
    </tr>
</table>

+ WEB资源
    
    1 资源概述：所有能够提供WEB内容的东西都是WEB资源
    
    WEB服务器是WEB资源的宿主,而WEB资源是WEB内容的源头
        
    静态资源：静态文件等,动态资源：根据需要生成的软件程序

    2 资源名称：URI与URL/URN
    
    1.名称含义
    
        URI 统一资源表示符 Uniform Resource Identifier
        URL 统一资源定位符 Uniform Resource Location
        URN 统一资源名       Uniform Resource Name
    
    2.关系：服务器资源名称称为URI，URI的两种实现形式，分别是URL(最常见)和URN（实验阶段）
    
    URL描述了一台特定服务器上某资源的特定位置，明确定义了如何从一个精确、固定的位置获取资源。与位置相关
    
    URN作为特定内容的唯一名称，与资源所在地无关。使用这些位置无关资源可以将资源四处搬移。使用统一名称来访问，需要一个支撑架构来解析资源的位置
    
    3.URL格式：三部分（协议、服务器和本地资源）
    
    方案：指明访问资源所使用的协议类型 HTTP://等

    服务器的因特网地址: www.baidu.com、 192.168.0.1等

    其余指定WEB服务器上的某个资源: /、/z/shibada/zhuanti.html等

    3 资源类型：WEB传输对象的MIME类型
    
    MIME（Miltipurpose Internet Mail Extension, 多用途因特网邮件扩展）最初解决不同电子邮件系统之间转移报文时存在的问题
    
    HTTP采纳MIME用以描述并标记多媒体内容，WEB服务器为所有的HTTP对象附加一个MIME类型(content-type)
    
    MIME类型是一种文本标记，表示一种主要的对象类型和一个特定的子类型，中间由一条斜杠来分隔
    
    常见类型
        
        HTML格式文本文件  text/html
        普通ASCII文本文件  text/plain
        JPEG版本图片文件   image/jpeg
        GIF格式的图片文件  image/gif
        APPLE的Quicktime电影 video/quicktime
        微软PPT演示文件     application/vnd.ms-powerpoint

+ HTTP事务
    
    1 HTTP事务 一个HTTP事务是由一条请求命令（从客户端发往服务器端）和一个响应结果（从服务器端发往客户端）组成的

    2 方法：请求命令称为HTTP方法。告诉服务器要执行什么动作

    3 状态码：HTTP报文返回携带一个状态码，告知客户端执行结果

    4 常见方法： 
        
        GET         请求服务器向客户端发送命名资源
        PUT         将来自客户端的数据存储到一个命名的服务器资源中去
        POST        将客户端数据发送到一个服务器网关应用程序
        HEAD        仅发送命名响应中的HTTP资源
        TRACE       对可能经过代理服务器传送到服务器上的报文进行追踪
        DELETE      从服务器中删除命名资源
        OPTIONS     决定可以在服务器上执行哪些方法

    5 常见状态码（状态码 原因短语 含义）
        
        1XX 信息性状态码
        2XX 成功状态码
        3XX 重定向状态码
        4XX 客户端错误状态码
        5XX 服务器错误状态码
        
        100 Continue
        200 OK 
        302 Found
        304 Not Modified
        403 Forbidden
        404 Not Found
        500 Internal Server Error
        502 Bad Gateway
        503 Service Unavailable

    6 WEB页面可以包含多个对象，一个页面通常不是单个资源，而是一组资源的集合。复合WEB页面要为每个嵌入式资源使用以一个单独的HTTP事务

+ HTTP报文

    1 通用报文格式：请求报文和响应报文
    
    起始行：报文的第一行就是起始行，请求报文中说明要做什么，响应报文中说明出现了什么情况
    
    请求报文起始行：包含一个方法和一个请求URL。方法描述了服务器应该执行的操作，请求URL描述了要对哪个资源执行这个方法。请求行还包含HTTP版本号。
    
    响应报文起始行：包含了响应报文使用的HTTP版本、数字状态码、以及描述操作状态的文本形式的原因短语。
    
    首部字段：起始行后面有零个或多个首部字段。每个首部字段都包含一个名字一个值（Key-Value形式），以冒号隔开。首部以空行结束，添加一个首部字段和添加新行一样简单。
    
    主体：空行之后就是主体，包含了所有类型的数据。请求主体包括了发给WEB服务器的数据，响应主体装载了要返回给客户端的数据。起始行和首部都是文本形式且都是格式化的，而主体则可以是任意类型。

    2 首部字段与方法配合工作，决定了客户端和服务器能做什么事情。格式前面已经指明，首部字段域的名字与大小写无关。在请求和响应的报文中都可以用首部来提供信息。可以将首部分为五个主要类型。

    普通首部：客户端和服务器都可以使用的通用首部
    
    举例：Connection,Date,MIME-Version,Trailer,Transfor-Encoding,Update,Via,<strong>Cache-Control</strong>,Pragma

    请求首部：只在请求报文中有意义，用于说明是谁或什么在发送请求、请求原子何处、或者客户端喜好及能力。

    举例：Host,From,Accept,Accept-*,Except,If-Match,If-*,<strong>If-Modified-Since,Cookie</strong>,Max-Forward

    响应首部：响应首部为客户端发送额外信息，比如是谁在发送响应、响应者的功能，甚至与响应相关的一些特殊指令。

    举例：Age,Server,<strong>Set-Cookie</strong>,Title,Warning,Vary,Accept-Ranges,WWW-Authenticate

    实体首部：用来描述HTTP报文的负荷，它可以告知报文接收者它在对什么进行处理。请求和响应报文中都可能出现。

    举例：Allow,Location,Content-Type,Content-Encoding,Content-*,<strong>Expries,Etag,Last-Modified</strong>

    扩展首部：非标准首部，开发者自建。

    注：具体报文可以对比前面TCPdump抓包结果。具体后面有文章详细剖析。

* * *

###参考资料###

+ [HTTP权威指南](http://book.douban.com/subject/10746113/)
+ [HTTP协议详解](http://vdisk.weibo.com/s/bh3V-)

---
title: HTTP报文与抓包分析
date: '2012-12-16'
categories: Notes
tags: ['HTTP','Note']
---


###主要内容###
这次的内容主要是回顾与总计与总结关于HTTP报文相关知识，并对抓包方法与工具进行总结和介绍。

![http报文整体]({{urls.media}}/HTTP报文总体.jpeg)
* * *

###HTTP报文###

+ 基础知识回顾 

分类与组成部分：请求报文与响应报文：报文均由请求行+首部+可选的实体的主体部分

![http报文整体]({{urls.media}}/HTTP报文.jpeg)

由于首部字段居多，而且针对不同功能有不同用法，比如有些会针对缓存设计，有些针对代理设计等等。这些在后面具体部分还会在详细深入。这里只是简单提出。本部分会总计部分跟缓存相关的首部。这是我过去接触比较多的一些。

+ 常见报文展示

使用wireshark工具，抓取常见HTTP请求回应包，可以对之前的总结有更直观的认识。

<strong>GET请求示例</strong>

![GET请求]({{urls.media}}/http_get.png)

<strong>POST请求示例</strong>

![POST请求]({{urls.media}}/http_post.png)

<strong>OK响应示例</strong>

![200响应]({{urls.media}}/http_ok.png)

<strong>报文流</strong>

![Request与Response构成报文流]({{urls.media}}/req_res.png)

图中标出了两个报文流，这也可以看出HTTP<strong>请求响应</strong>模式

+ 关于GET和POST

Http协议定义了很多与服务器交互的方法，常用的有七个，最基本的有4个，分别是GET,POST,PUT,DELETE。 一个URL地址用于描述一个网络上的资源，而HTTP中的GET, POST, PUT, DELETE就对应着对这个资源的查，改，增，删4个操作。类似数据库的操作。 最常见的就是GET和POST了。GET一般用于获取/查询资源信息，而POST一般用于更新资源信息。

GET和POST的基本区别：

    GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456.  POST方法是把提交的数据放在HTTP包的Body中.
    GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.
    GET和POST取变量值的方法不同。
    GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码。


+ 缓存相关首部

**Request相关**

<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>Cache-Control: max-age=0</td>
            <td >以秒为单位</td>
        </tr>
        <tr>
            <td>If-Modified-Since: Mon, 19 Nov 2012 08:38:01 GMT</td>
            <td>缓存文件的最后修改时间</td>
        </tr>
        <tr>
            <td>If-None-Match: &quot;0693f67a67cc1:0&quot;</td>
            <td>缓存文件的Etag值</td>
        </tr>
        <tr>
            <td>Cache-Control: no-cache</td>
            <td>不使用缓存</td>
        </tr>
        <tr>
            <td>Pragma: no-cache</td>
            <td>不使用缓存</td>
        </tr>
    </tbody>
</table>

**Response**相关

<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>Cache-Control: public</td>
            <td>响应被缓存，并且在多用户间共享</td>
        </tr>
        <tr>
            <td>Cache-Control: private</td>
            <td>响应只能作为私有缓存，不能在用户之间共享</td>
        </tr>
        <tr>
            <td>Cache-Control:no-cache</td>
            <td >提醒浏览器要从服务器提取文档进行验证<td>
        </tr>
        <tr>
            <td>Cache-Control:no-store</td>
            <td >绝对禁止缓存(用于机密，敏感文件)</td>
        </tr>
        <tr>
            <td>Cache-Control: max-age=60</td>
            <td>60秒之后缓存过期(相对时间)</td>
        </tr>
        <tr>
            <td >Date: Mon, 19 Nov 2012 08:39:00 GMT</td>
            <td >当前response发送的时间</td>
        </tr>
        <tr>
            <td >Expires: Mon, 19 Nov 2012 08:40:01 GMT</td>
            <td >缓存过期的时间（绝对时间）</td>
        </tr>
        <tr>
            <td >Last-Modified: Mon, 19 Nov 2012 08:38:01 GMT</td>
            <td >服务器端文件的最后修改时间</td>
        </tr>
        <tr>
            <td >ETag: &quot;20b1add7ec1cd1:0&quot;</td>
            <td >服务器端文件的Etag值</td>
        </tr>
    </tbody>
</table>

* * *

###HTTP抓包###

+ 浏览器抓包

不同浏览器都有相应的抓包分析工具，可以对HTTP连接进行简单的查看。最经典最常用的有Firefox的Firebug，Chrome浏览器直接审查元素等方式。

Firebug 简介

Firebug集HTML查看和编辑、Javascript控制台、网络状况监视于一体，可以说是开发人员必备扩展之一。Firebug从各个不同的角度剖析 Web页面内部的细节层面，给Web开发者带来很大的便利。安装使用简单，直接在Firefox浏览器的加载项中安装使用即可。

Firebug 抓包

使用Firebug抓包，访问百度首页两次，比较访问其中时间数据，在看看具体请求响应报文。这些都是Firebug工具很容易实现的功能。

<strong>两次访问百度</strong>

![baidu]({{urls.media}}/baidu.png)

<strong>第一次访问时间数据统计</strong>

![baidu_first]({{urls.media}}/baidu_first.png)

<strong>第二次访问时间数据统计</strong>

![baidu_second]({{urls.media}}/baidu_second.png)

<strong>访问百度请求响应数据包</strong>

![baidu_http]({{urls.media}}/baidu_http.png)

+ 其他工具抓包

工具类主要是wireshark和tcpdump。这是两个强大的工具。前面举例中已经使用了wireshark抓包。这里再介绍下如果需要抓HTTP包，主要是过滤规则的设置。对于HTTP包，wireshark常见的过滤规则有：

http 所有http包

http.connection 过滤出应用程序的http请求和返回

http.date 仅仅显示返回包

http.host 应用程序和设备消息查询的请求包

http.referer 应用程序的请求包

http.request 应用程序和设备的请求包

http.response 应用程序的返回包

http.server 同上（服务器返回的包）

下图是比较完整的HTTP过滤规则【因为一张图无法全部截下，遗漏最后几个，可以在wireshark中查出】：

![wirshark]({{urls.media}}/wireshark-filter.png)

分析HTTP报文相对来说，wireshark过滤规则比tcpdump更加灵活，能够针对具体的首部、类型、组成等来进行分析。至于tcpdump在后面关于分析连接中再详细介绍。
* * *
###参考资料###
+ [HTTP权威指南](http://book.douban.com/subject/10746113/)
+ [HTTP之缓存](http://www.cnblogs.com/TankXiao/archive/2012/11/28/2793365.html)

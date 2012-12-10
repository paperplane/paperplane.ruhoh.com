---
title: URL原理及设计
date: '2012-12-09'
description: 
categories: Notes
tags: ['HTTP','Note']
---

###URL基础知识###
+ **URL概念**

    URL是因特网资源的标准化名称。URL为用户及浏览器提供找到信息所需要的所有条件：它定义了用户所需的特定资源，它位于何处以及如何获取它。

    URI是一类通用的资源标示符，而URL只是它的子集。URI由URL和URN构成，URL通过描述资源的位置来标识资源，而URN则是通过名字来识别资源，与位置无关。

    HTTP规范将更通用的概念URI作为其资源标示符，但实际上，HTTP应用程序处理的只是URI的URL子集。

+ **通用格式**
   
    \<scheme\>://\<user\>:\<password\>@\<host\>:\<port\>/\<path\>;\<parameters\>?\<query\>#\<frag\>
   
    几乎没有URL能够包含所有组件，一般都只包含最重要的三部分：方案(\<scheme\>)、主机(\<host\>)和路径(\<path\>)

+ **URL语法**

通用URL组件

<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>组件</td>
            <td>描述</td>
            <td>默认值</td>
            <td>举例及说明</td>
        </tr>
        <tr>
            <td>方案</td>
            <td>访问服务器获取资源时使用什么协议，以第一个字母符号开始，由第一个&quot;:&quot;结束，大小写无关</td>
            <td>无</td>
            <td>HTTP:HTTPS:MAILTO:FTP:RTSP:RTSPU:FILE:NEWS:TELNET</td>
        </tr>
        <tr>
            <td>用户名和密码</td>
            <td>用户名：某些方案访问资源时需要资源名<br/>密码：用户名后面可能要包含的密码，中间用&quot;:&quot;分隔</td>
            <td>匿名</td>
            <td><a href="ftp://ftp.prep.ai.mit.edu/pub/gnu">ftp://ftp.prep.ai.mit.edu/pub/gnu</a><br/><div><a href="ftp://ftp.prep.ai.mit.edu/pub/gnu">ftp://anonymous@ftp.prep.ai.mit.edu/pub/gnu</a></div><div><a href="ftp://ftp.prep.ai.mit.edu/pub/gnu">ftp://anonymous:passwd@ftp.prep.ai.mit.edu/pub/gnu</a></div><div><a href="http://user:passwd@www.test.com/">http://user:passwd@www.test.com</a></div></td>
        </tr>
        <tr>
            <td>主机名和端口</td>
            <td>主机：资源宿主服务器的主机名或者IP<br/>端口：资源宿主服务器正在监听的端口,解决了在哪台机器上装载资源和在机器的什么地方访问资源的两组信息</td>
            <td>每个方案特有</td>
            <td><a href="http://www.baidu.com/index.html">http://www.baidu.com:80/index.html</a><br/><a href="http://202.112.128.51/index.html">http://202.112.128.51:80/index.html</a></td>
        </tr>
        <tr>
            <td>路径</td>
            <td>服务器上资源的本地名，由一个&quot;/&quot;将其与其他前面的组件分隔开。路径组件语法与服务器和方案有关</td>
            <td>无</td>
            <td><a href="http://www.joes-hardware.com/seasonal/index-fall.html">http://www.joes-hardware.com/seasonal/index-fall.html</a>这其中<a href="http://www.joes-hardware.com/seasonal/index-fall.html">/seasonal/index-fall.html</a>则为路径</td>
        </tr>
        <tr>
            <td>参数</td>
            <td>某些方案会用这个组件来指定输入参数。参数为名/值对。URL中可包含多个参数字段，它们之间以及与路径的其余部分用&quot;;&quot;分隔</td>
            <td>无</td><td><a href="ftp://prep.ai.mit.edu/gnu;type=d">ftp://prep.ai.mit.edu/gnu;type=d</a>中<a href="ftp://prep.ai.mit.edu/gnu;type=d">type=d</a>为参数。<br/><a href="http://joes-hardware.com/hammers;sale=false/index.html;graphics=true">http://joes-hardware.com/hammers;sale=false/index.html;graphics=true</a><br/>路径组件每段都提供自己的参数</td>
        </tr>
        <tr>
            <td>查询</td>
            <td>某些方案会有查询字符串来传递参数以激活应用程序（比如数据库、公告板、搜索引擎以及其他因特网网关）。查询组件的内容没有通用格式。用&quot;?&quot;将其与其余部分隔开。</td>
            <td>无</td><td><a href="http://joes-hardware.com/inventory-check.cgi?item=12731&amp;amp;color=blue&amp;amp;size=large">http://joes-hardware.com/inventory-check.cgi?item=12731&amp;color=blue&amp;size=large</a> 多个查询条件使用&quot;&amp;&quot;隔开<br/></td>
        </tr>
        <tr>
            <td>片段</td>
            <td>一小片或一部分资源的名字。引用对象时，不会讲frag字段传送给服务器，这个字段在客户端内部使用。通过&quot;#&quot;将其与URL其余部分隔开</td>
            <td>无</td>
            <td><a href="http://twitter.github.com/bootstrap/scaffolding.html#gridSystem">http://twitter.github.com/bootstrap/scaffolding.html#gridSystem</a><br/>片段<a href="http://twitter.github.com/bootstrap/scaffolding.html#gridSystem">gridSystem</a>引用了<a href="http://twitter.github.com/bootstrap/scaffolding.html#gridSystem">/scaffolding.html</a>页面的一部分，这部分名为<a href="http://twitter.github.com/bootstrap/scaffolding.html#gridSystem">gridSystem</a></td>
        </tr>
    </tbody>
</table>
    
注意：HTTP服务器只处理整个对象，而不是对象的片段，客户端不能讲片段传给服务器。浏览器从服务器获取整个资源后，会根据片段显示感兴趣的那部分资源

常见方案URL格式与举例

<table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>方案</td>
            <td>格式</td>
            <td>举例</td>
        </tr>
        <tr>
            <td>HTTP</td>
            <td>http://&lt;host&gt;:&lt;port&gt;/&lt;path&gt;?&lt;query&gt;#&lt;frag&gt;</td>
            <td><a href="http://www.joes-hardware.com/index.html">http://www.joes-hardware.com/index.html</a> <br/><a href="http://www.joes-hardware.com/index.com">http://www.joes-hardware.com:80/index.com</a> </td>
        </tr>
        <tr>
            <td>HTTPS</td>
            <td><div>https://&lt;host&gt;:&lt;port&gt;/&lt;path&gt;?&lt;query&gt;#&lt;frag&gt;</div></td>
            <td><a href="https://www.joes-hardware.com/secure.html">https://www.joes-hardware.com/secure.html</a> </td>
        </tr>
        <tr>
            <td>MAILTO</td>
            <td>mailto:&lt;RFC-822-addr-spec&gt;</td>
            <td><a href="mailto:jql.buaa@gmail.com">mailto:jql.buaa@gmail.com</a> </td>
        </tr>
        <tr>
            <td>FTP</td>
            <td>ftp://&lt;user&gt;:&lt;password&gt;@&lt;host&gt;:&lt;port&gt;/&lt;path&gt;;&lt;params&gt;</td>
            <td><a href="ftp://anonymore:joe%40joes@prep.ai.mit.edu/pub/gnu/">ftp://anonymore:joe%40joes@prep.ai.mit.edu:21/pub/gnu/</a> </td>
        </tr>
        <tr>
            <td>RTSP<br/>RTSPU</td>
            <td>rtsp://&lt;user&gt;:&lt;password&gt;@&lt;host&gt;:&lt;port&gt;/&lt;path&gt;<br/><div>rtspu://&lt;user&gt;:&lt;password&gt;@&lt;host&gt;:&lt;port&gt;/&lt;path&gt;</div></td>
            <td>rtsp://www.joes-hardware.com:554/interview/cto_video<br/><br/></td>
        </tr>
        <tr>
            <td>FILE</td>
            <td>file://&lt;host&gt;/&lt;path&gt;</td>
            <td><a href="file://OFFICE-FS/politicies/casual-fridays.doc">file://OFFICE-FS/politicies/casual-fridays.doc</a> </td>
        </tr>
        <tr>
            <td>NEWS</td>
            <td>news://&lt;newsgroup&gt; news://&lt;news-article-id&gt;</td>
            <td>news:rec.arts.startrek</td>
        </tr>
        <tr>
            <td>TELNET</td>
            <td>telnet://&lt;user&gt;:&lt;password&gt;@&lt;host&gt;:&lt;port&gt;/</td>
            <td>telnet://slurp:webhound@joes-hardware.com:23/</td>
        </tr>
    </tbody>
</table>

+ **适用协议**

    URL是一个标准化概念，**URL并不只用于HTTP协议**。由方案的概念可以看出，它还可以http(s)、mailto、ftp、rtsp、rtspu、file、news、telnet

+ **相对URL**

    相对路径做对比：URL本身就是路径的概念，相对URL与相对路径可完全类比。而优势也正如使用相对路径一样。

    相对URL的解析:将相对URL转换成绝对URL的过程。其原理就是将基础和相对URL划分成组件组合成新的绝对URL。

+ **字符表与编码机制**
    
    1 字符表

    US-ASCII字符集**集成转义序列**，这样可包含任意的二进制数据和其他变体字符等

    2 编码规则

    为避开安全字符集表示法带来的限制，设计转义编码机制。

    **转义法**：一个"%",后面跟着两个表示字符ASCII码的十六进制数

    <table class="table table-bordered table-striped table-condensed">
    <tbody>
        <tr>
            <td>字符</td>
            <td>ASCII</td>
            <td>示例URL</td>
        </tr>
        <tr>
            <td>~</td>
            <td>0x7E</td>
            <td><a href="http://www.joes-hardware.com/%7Ejoe">http://www.joes-hardware.com/%7Ejoe</a></td>
        </tr>
        <tr>
            <td>空格</td>
            <td>0x20</td><td><a href="http://www.joes-hardware.com/more%20tools.html">http://www.joes-hardware.com/more%20tools.html</a></td>
        </tr>
        <tr>
            <td>%</td>
            <td>0x25</td>
            <td><a href="http://www.joes-hardware.com/100%25satisfaction.html">http://www.joes-hardware.com/100%25satisfaction.html</a></td>
        </tr>
    </tbody>
    </table>

    3 受限字符

    **% / . .. # ? ; : $ + @ & = {} | \^ ~ [] ` <> "" 0x00-0x1F 0x7E >0x7E**

+ **长度上限**

    URL的最大长度是多少？W3C的HTTP协议并没有限定，然而，在实际应用中，经过试验，不同浏览器和Web服务器有不同的约定：

        IE的URL长度上限是2083字节，其中纯路径部分不能超过2048字节。
        Firefox浏览器的地址栏中超过65536字符后就不再显示。
        Safari浏览器一致测试到80000字符还工作得好好的。
        Opera浏览器测试到190000字符的时候，还正常工作。
        Apache Web服务器在接收到大约4000字符长的URL时候产生"413 Entity Too Large"错误。
        IIS默认接收的最大URL是16384字符。

+ **自动扩展URL**

    扩展：基于**主机名**和基于**历史**

    这种机制依赖浏览器的实现

* * *

###URL设计原则###

+ **一个URL必须唯一地，永久地代表一个在线对象**

    URL 的最基本的使命是唯一地代表 Internet 上的一个对象，URL 必须和 Internet 上的对象一对一匹配。

+ **尽可能用户友好**

    这是 URL 设计的根本，你的 URL 应该为最终用户而设计。保持 URL 友好的一个好办法是在保证可读性的同时让它尽可能短。短地址的崛起虽然能很好地使地址变短，却在一定程度下失去了可读性。

+ **保持一致性**

    站点内的所有 URL 必须保持一致的格式和结构，这样可以为用户带来信任感，如果你必须更改 URL 格式和结构，需要使用 HTTP 301 机制。

+ **可预测的URL**

    这也是 URL 一致性的一个表现，如果你的 URL 拥有很好的一致性，用户可以根据 URL 猜测别的内容的 URL，假如 /events/2010/01 指向 2010 年 1 月份的日程内容，那
    
    /events/2009/01 应当指向 2009 年 1 月的日程。 
    
    /events/2010 应当指向 2010 年全年的日程。 
    
    /events/2010/01/21 应当指向2010年1月21日的日程。

+ **细节注意点**

    1 URL不应包含.html,aspx,cfm一类的后缀

    这类信息对最终用户是没有意义的，却占了额外的空间，一个例外是.atom,.rss,.json一类的特殊地址，这类地址是有特别的意义的。译者注：在某些虚拟主机式Web服务器，这种做法未必现实。

    2 URL不应包含WWW部分

    WWW部分并不包含任何意义，是一个额外的负担，不友好。可以使用HTTP 301机制，将www.domain.com定向到domain.com。

* * *

###ShortURL设计###

+ 短地址使用崛起

    如今的URL已经变得越来越长，主机名越来越长，各种应用导致参数也越来越多等等，尤其是以微博为主要的崛起，导致短地址迅速普及。短地址的出现能够很好地缩短地址，更好支持自定义，添加备注方便管理，此外对于数据统计：访问时段、地区、来路等统计以及统计信息，方便收藏统计。

+ 常见短地址服务(仅举部分实例)
    
    著名的[tinyul](http://tinyurl.com/)
    
    [Doiop](http://doiop.com/)
    
    [0gg](http://0.gg/)
    
    [google](http://goo.gl)
    
    [Fly2](http://www.fly2.ws/)
    
    国内[网易短地址](http://126.am/), 并提供有API文档： [API文档](http://126.am/client/api_register_new.jsp)

+ 短地址常用算法

    算法这部分主要来自[百度百科](http://baike.baidu.com/view/5712914.html)介绍，其他能搜索到的算法主要也就是这两种，网上有第一种算法的实现。关于算法实现，以后会补上个人实现。[另一汇总地址](http://blog.xe28.tk/archives/188/)

    算法一 [实现](http://blog.csdn.net/xyz_lmn/article/details/8057270)   

        1)将长网址md5生成32位签名串,分为4段, 每段8个字节;   
        2)对这四段循环处理, 取8个字节, 将他看成16进制串与0x3fffffff(30位1)与操作, 即超过30位的忽略处理;   
        3)这30位分成6段, 每5位的数字作为字母表的索引取得特定字符, 依次进行获得6位字符串;   
        4)总的md5串可以获得4个6位串; 取里面的任意一个就可作为这个长url的短url地址;

    算法二   
        
        a-zA-Z0-9 这64位取6位组合,可产生500多亿个组合数量.把数字和字符组合做一定的映射,就可以产生唯一的字符串,如第62个组合就是aaaaa9,第63个组合就是aaaaba,再利用洗牌算法，把原字符串打乱后保存，那么对应位置的组合字符串就会是无序的组合。     
        
        把长网址存入数据库,取返回的id,找出对应的字符串,例如返回ID为1，那么对应上面的字符串组合就是bbb,同理 ID为2时，字符串组合为bba,依次类推,直至到达64种组合后才会出现重复的可能，所以如果用上面的62个字符，任意取6个字符组合成字符串的话，你的数据存量达到500多亿后才会出现重复的可能。

+ 短地址设计原则
   
    短地址与长地址的key-value匹配记录

    同步点即何时做同步控制。

* * *

###URL展望###

+ **URL的局限性**

    URL的局限性在于无法根据名称定位。URL表示的实际的地址，而不是准确的名字。这就意味着URL会告诉你资源此时处于什么位置，它会为你提供特定端口上特定服务器的名字，告诉你在何处可以找到资源。这种方案的特点在于如果资源被移走，URL就不再有效。

+ **PURL的出现**

    永久统一资源定位符 persistent uniform resource locators

    基本思想是在搜索资源的过程中引入中间层，通过中间资源定位符服务器对资源的实际URL进行登记和更新。客户端可以向定位符请求一个永久URL，定位符可以以一个资源作为响应，将客户端重定向到资源的实际URL上去。

+ **URN是未来？**

    URN：URL的一种更新形式，统一资源名称(URN, Uniform Resource Name)不依赖于位置，并且有可能减少失效连接的个数。但是其流行还需假以时日，因为它需要更精密软件的支持。它的设计在于唯一性与持久性，即使资源不在存在或不再使用。目前实现上还有待进一步规范，所以尚需时日。至于以后是不是方向，起码提供了一种实现思路。主要实现难点，在于需要一个支撑架构来解析资源的位置。

* * *

###参考资料###

+ [URL设计原则](http://www.comsharp.com/GetKnowledge/zh-CN/CMS_K1007.aspx), [英文原文](http://css-tricks.com/guidelines-for-uri-design/)
+ [HTTP权威指南](http://book.douban.com/subject/10746113/) 

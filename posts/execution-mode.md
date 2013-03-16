---
title: Execution Mode
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##执行模式--Execution Mode##

之前定义了主机列表和任务列表，fabric根据主机列表和任务列表就可以执行任务，下面就是分析其执行策略及其他相关细节。由fab传入相应的参数就可以实现之前的多个主机执行多个任务的目标。现在先来看默认执行策略是什么样的。

***

####执行策略####

fabric函数默认是单个、顺序执行方式，当然它也提供并行执行。默认执行行为：

1.创建一个任务列表，这些任务就是fab命令的参数，并且保持fab传参的顺序

2.对于每个任务，会通过特定方式创建特定主机列表

3.任务按照顺序执行，每个人物会在指定特定主机上执行一次

4.对于没有指定主机的任务，默认是本地任务，仅执行一次

    from fabric.api import run, env
    env.hosts = ['host1', 'host2']
    def taskA():    
         run('ls')
    def taskB():    
         run('whoami')
    $ fab taskA taskB

执行顺序就会是：

    taskA on host1
    taskA on host2
    taskB on host1
    taskB on host2
    
***

####智能执行####

在1.3版本之前，fabric只能针对特定任务指定特定主机列表，或者全局使用相同主机列表，若一个任务使用role1,而另一个任务使用role2，而定义了新的函数来执行这两个任务，role1，和role2不尽相同。则此时需要使用execute 来执行。

举例如下：

    from fabric.api import run, roles

    env.roledefs = {
        'db': ['db1', 'db2'],
    'web': ['web1', 'web2', 'web3']
    }
    @roles('db')
    def migrate():
        pass
    @roles('web')
    def update():
        pass

定义新函数实现在同一任务调用时用db来migrate,而用web来update

    def deploy():
        execute(migrate)
        execute(update)

***

####建立连接####

fab 自身并不与远程主机建立实际连接。而是对于每一个需要执行的任务的主机列表的每一个主机，该任务的env 变量中 env.host_string 指定正确的值（主机）。env.host_string 指定当前host string,当需要网络的函数执行时 fabric 用来决定是建立连接还是在共享字典（映射 SSH 连接对象）里查找并重用。像run/put这样的操作使用env.host_string作为一个共享字典的查找表来映射host strings和ssh 连接对象。

需要使用fabric作为类库来使用时需要手动设定须连接的对象 。这在类库使用部分有详细介绍几种使用方式。

fabric 建立的连接是一种懒惰连接，任务相关，不到万不得已不会建立，而连接的释放并不会随着缓存的消失而关闭，直到任务执行结束或显示调用disconnect_all 去关闭连接。

对于多连接中可能遇到的某些 bad hosts，可以显式设置env.skip_bad_hosts以及超时时间和连接次数等.

    from fabric.api import *
    @hosts('host1')
    def clean_and_upload():
        local('find assets/ -name "*.DS_Store" -exec rm '{}' \;')
        local('tar czf /tmp/assets.tgz assets/')
        put('/tmp/assets.tgz', '/tmp/assets.tgz')
        with cd('/var/www/myapp/'):
            run('tar xzf /tmp/assets.tgz')

1.  两个local调用不会建立任何网络连接

2.  put向连接缓存中要求同host1建立连接

3.  连接缓存中没有该连接的host string,则SSH创建新的连接，返回给put

4.  put通过该连接上传文件

5.  最后，run询问连接缓存同样的host string,由于已经有了，则该连接被重用。

这就是连接过程，直到执行完毕。

***

####密码管理####

fabric在内存保留两层缓存来帮助记住login和sudo的密码，这避免了当多个系统密码相同时不断重复输入密码。

第一层是默认简单的密码缓存，env.password。这个环境变量保存一个单一的当当前特定主机（host_string）需要密码时会尝试的密码。

第二层env.passwords保存了最近输入的每一个不同的user/host/port组合的密码，它是一个per-user/per-host缓存。由于这个缓存，在同一个会话中连接多个不同user或者host每个只需要一个单一密码。

根据你的配置和你的回话连接的主机数量，你也许会发现任一或全部env vars会比较有用。当然，fabric会自动填写相关值而不需要额外的设置。特别地，每一次密码prompt,用户敲入的密码都会被用来更新当前连接的host_string的两个缓存的值。

***

####并行执行####

fabric默认的执行模式是串行执行，正如我们在前面所看到的一样。fabric同样提供并行执行功能，目前是通过python的multiprocessing模块来实现。它为每一个主机和任务组合创建一个线程，有选择地使用sliding window来阻止同时创建过多线程。

对于之前串行的例子，如果执行
    
    $ fab -P taskA taskB

执行结果就会是：

    taskA on host1,host2
    taskB on host1,host2

怎样使用：

装饰器：
    
    @parallel(另@serial default值)

命令行：

    -P（default为serial）

    from fabric.api import *
    @parallel
    def runs_in_parallel():
        pass
    $fab run_in_parallel
    Bubble Size

对于大量的主机列表，如果执行太多并发fabric进程会容易使机器宕机。为此，使用pool_size来限制fabric并发活动进程数量。默认pool_size为1.

使用方法:

    from fabric.api import *
    @parallel(pool_size=5)
    def heavy_task():
        # lots of heavy local lifting or lots of IO here

或者通过-z选项来指定：

    $ fab -P -z 5 heavy_task

Linewise output

fabric默认输出终端的方式是type_by_type,为了支持远程交互。这会导致在并行执行时糟糕的输出，因为多线程可能同时向终端标准输出流。所以在并行执行时，fabric默认是linewise output。这是并行和输出的折中，虽然fabric远程交互特点大打折扣，但是相对那些在并行时没有很好处理输出应经好很多。

现在并行时虽然不同执行行与行之间交叉，但可以通过每行的host string前缀处理 。就是很多命令交叉着出现

    [host1]out:xxx
    [host2]out:yyy
    [host3]run:zzz
    [host2]out:ppp

这样很乱，如果需要提取每一台主机返回的信息，需要进行字符串处理。

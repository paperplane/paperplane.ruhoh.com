---
title: Task List
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##任务列表--Task List##

有了主机列表，我们能指定任务执行的主机，下面就是对需要执行的任务的定义。在当前版本（1.1之后）fabric引进两种不同的任务定义方法。现在分别称之为新风格和经典风格。它们在判断fabfile中什么对象才是fabric的任务时（task）方式不同：

新风格认为Task的实例或者它的子类为任务，以及import的模块，而且允许创建嵌套的命名空间。

经典风格认为所有public的可调用对象（包括functions,classes等等），而且仅仅认为是fabfile中的对象才行而没有递归import的模块。

注意：这两种定义方式是相互排斥的，如果fabric发现任何新风格的任务对象在fabfile或它import的模块中时，它就会认为你已经使用这种新风格的任务声明，而不会考虑任何没有Task声明的对象。如果没有新风格任务发现，它会使用默认经典风格。可以通过命令fab --list查看可用任务名及说明。

***

####新风格任务####

新风格任务对面行对象特性和命名空间有很好的支持。尤其是面向对象的继承和多态特性，对代码的复用极其重要。通过引进Task，有两种方式来定义新任务。

定义常规的模块级别的函数并带有装饰器@task，这会直接将该函数转化为Task子类。该函数名会被作为任务名。

Task子类，定义run方法，模块级别实例化你的子类。实例名称的属性会被作为任务名，如果省略实例的变量名会被用作任务名。

两种定义：
    
    from fabric.api import task, run
    @task
    def mytask():
        run("a command")
    
以及：

    class MyTask(Task):
        name = "deploy"
        def run(self, environment, domain="whatever.com"):
            run("git clone foo")
            sudo("service apache2 restart")
    instance = MyTask()

下面分别详细介绍上述两种定义。@task参数：

task_class：被装饰的函数的类，Task的子类，默认是WrappedCallableTask.

aliases：一个可迭代的string names,被用来作为该函数的别名。

alias：跟aliases类似，只是只有一个string值而不是可迭代对象。如果aliases同时存在，前者优先。

default：布尔值决定这个被装饰函数时候同时被包括在模块内作为一个任务名。若含有default=True的函数，fab不含参数时默认执行该任务。

上述task子类完全等同于下面定义：

    @task
    def deploy(environment, domain="whatever.com"):
        run("git clone foo")
        sudo("service apache2 restart")

所以一般结合这两种定义方式：

    from fabric.api import task
    from fabric.tasks import Task

    class CustomTask(Task):
        def __init__(self, func, myarg, *args, **kwargs):
            super(CustomTask, self).__init__(\*args, **kwargs)
            self.func = func
            self.myarg = myarg
        def run(self, *args, **kwargs):
            return self.func(\*args, **kwargs)
    @task(task_class=CustomTask, myarg='value', alias='at')
    def actual_task():
        pass

命名空间的支持类似python语法。仅举例说明：

有文件树结构如下：

    .
    ├── __init__.py
    ├── db
    │   ├── __init__.py
    │   └── migrations.py
    └── lb.py

外层__init__.py:

    import lb
    import db
    @task
    def deploy():
        ...
    @task
    def compress():
        ...
    lb.py 内容：
    @task
    def add_backend():
        ...

内层__init__.py:

    import migrations
    migrations.py内容：
    @task
    def list():
        ...
    @task
    def run():
        ...

最终任务列表：

    deploy
    compress
    lb.add_backend
    db.migrations.list
    db.migrations.run

***

####经典任务####

经典任务就是普通的python函数，当没有新风格任务发现时，fabric默认都是经典任务，注意一下情况：

可调用的函数名以_下划线开头，python认为是private的函数，fabric认为不是任务。

可调用的但是没有用到fabric自身的，fabric认为不是任务。

这里仅仅指可调用对象，而不是module, fabric经典风格不能import模块作为其他任务。

定义经典风格任务与定义python函数基本类似，主要使用到fabric的API即可。这里就不多说，但注意以下这种import情况：

原来写法：

    from urllib import urlopen
    from fabric.api import run
    def webservice_read():
        objects = urlopen('http://my/web/service/?foo=bar').read().split()
        print(objects)

改进写法：

    import urllib
    from fabric.api import run
    def webservice_read():
        objects = urllib.urlopen('http://my/web/service/?foo=bar').read().split()
        print(objects)

两种都没错只不过第一种写法fab会认为urlopen也是一个任务名，所以很可能被误用。

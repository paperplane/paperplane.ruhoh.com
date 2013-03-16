---
title: Fabric API Full List
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##Fabric API Full List##

####Core API####

核心API主要有七类:带颜色的输出类(color output),上下文管理类(context 
managers), 装饰器类(decorators), 网络类(network), 操作类(oprations), 任务类(tasks), 工具类(utils)。 

#####Color Output#####

每一个包含这个模块的函数返回String带有颜色。比如: 

    From fabric.api import green,red 
    Print (red("This sentence is red, except for " + green("these words, which are green") + "."))

共包括以下：

    fabric.colors.blue(text, bold=False)
    fabric.colors.cyan(text, bold=False)
    fabric.colors.green(text, bold=False)
    fabric.colors.magenta(text, bold=False)
    fabric.colors.red(text, bold=False)
    fabric.colors.white(text, bold=False)
    fabric.colors.yellow(text, bold=False)

#####Context Managers#####

Context Managers使用都需要结合with语句。连续使用多个时可嵌套也可用逗号隔开连接使用。

举例如下：

    with cd('/path/to/app'):
        with prefix('workon myvenv'):
            run('./manage.py syncdb')
            run('./manage.py loaddata myfixture')

它等价于

    with cd('/path/to/app'), prefix('workon myvenv'):
        run('./manage.py syncdb')
        run('./manage.py loaddata myfixture')

注意此时在python2.5中的写法：with nested(cd('/path/to/app'), prefix('workon myvenv')):

此类包括：

    fabric.context_managers.cd(path) cd(远程主机更新工作目录) 

任何被包括在 with cd(path):代码块里的命令run/sudo/get/put 相当于执行"cd <path> && "

那么很明显它与 shell 命令cd的区别举例如下：

    with cd('/var/www'):
        run('ls') # Turns into "cd /var/www && ls"

比较

    run('cd /var/www')
    run('ls')

前者相当于执行:run(‘cd /var/www && ls’)

后者相当于执行:ls 时并没在/var/www 路径下，而是在默认路径$HOME路径下

cd 可嵌套:

    with cd('/var/www'):
        run('ls') # cd /var/www && ls
        with cd('website1'):
            run('ls') # cd /var/www/website1 && ls
    
fabric.context_managers.lcd(path) lcd(本地主机更新工作目录) 同 cd用法相同,只是它改变的的是本地工作目录,而 cd 改变的远程主机工作目录，所以它只能改变local的调用以及put/get的本地参数，它的默认路径与fabfile所在路径相关，由环境变量env.real_fabfile指定
    
目前，cd和lcd的实现视是通过改变环境变量env.cwd和env.lcwd实现的，所以如果要实现这个也可以通过环境变量来实现，但是不建议这么做。因为按照官方文档说明，将来这种实现方式可能要改。


fabric.context_managers.hide(\*groups) hide(将指定参数输出级别默
认设置为 False) 指定默认隐藏的输出级别 group是一个或多个之前output
指定的类别之一，执行时它会将这些输出类型置为False。
比如你不想看到[hostname]:run:xxxx,以及阻止标准输出和错误就可以用下面这样

    def my_task():
        with hide('running', 'stdout', 'stderr'):
            run('ls /var/www')
    fabric.context_managers.show(\*groups) show(将指定参数输出级别默

认设置为 False) 指定默认输出的输出级别 用法同 hide，作用刚好相反。默认是所有都输出，所以show的一个作用就是打开默认隐藏的debug。
fabric.context_managers.path(path, behavior='append') 默认设置为将参数path附加在系统/用户环境变量$PATH，即 "PATH=$PATH:<path> "指定 run/sudo 路径

behavior默认还有两个参数：

    prepend:将指定参数path在$PATH前置即：PATH=<path>:$PATH
    replace：将指定参数代替$PATH，即PATH=<path>

fabric.context_managers.prefix(command) prefix( 对 于 所 有 包括在其中的run/sudo 命令相当于再给定参数命令后加了&&)

这与cd用法相同，只是在嵌套调用时，它是直接在后面附加字符串，而不是cd中的修改字符串。大多数时候，会用它来export或alter环境变量。

    with prefix('workon myvenv'):
        run('./manage.py syncdb')

等价于执行#workon myvenv && ./manage.py syncdb

嵌套调用举例：

    with prefix('workon myenv'):
        run('ls')
        with prefix('source /some/script'):
            run('touch a_file')

结果是：

    $ workon myenv && ls
    $ workon myenv && source /some/script && touch a_file

和cd 是兼容的，结合使用举例：

    with cd('/path/to/app'):
        with prefix('workon myvenv'):
            run('./manage.py syncdb')
            run('./manage.py loaddata myfixture')

结果如下：

    $ cd /path/to/app && workon myvenv && ./manage.py syncdb
    $ cd /path/to/app && workon myvenv && ./manage.py loaddata myfixture

fabric.context_managers.settings(\*args, **kwargs) setting(嵌套的上下文管理器覆盖env 变量) 它有两个作用:

大多数情况下,它会暂时覆盖/更新任何提到的关键字的 env 变量的值。比 如 : with settings(user='foo'): 相 当 于 设 定env.user=‘foo’。出了这个代码块,setting 中的值就会失效。此时注意 clean_revert=True 对作用域的影响,若此时在代码块中重新更新 env 的值,则该值就被更新即使出了代码块直到下 次 更 新 。 

    # Before the block, env.parallel defaults to False, host_string to None
    with settings(parallel=True, host_string='myhost'):
        # env.parallel is True
        # env.host_string is 'myhost'
        env.host_string = 'otherhost'
        # env.host_string is now 'otherhost'
    # Outside the block:
    # * env.parallel is False again
    # * env.host_string is None again

另 外 , 它 还 能 对 env 变 量 中 未 提 到 的 关 键 字（即env变量中没有的，这些可能会是其他的上下文管理器） 进 行 指 定 值 , 比 如 :

    def my_task():
        with settings(
            hide('warnings', 'running', 'stdout', 'stderr'),
            warn_only=True
        ):
            if run('ls /etc/lsb-release'):
                return 'Ubuntu'
            elif run('ls /etc/redhat-release'):
                return 'RedHat'

最后就是clean_revert设置带来的变化注意对比前面例子：

    # Before the block, env.parallel defaults to False, host_string to None
    with settings(parallel=True, host_string='myhost', clean_revert=True):
        # env.parallel is True
        # env.host_string is 'myhost'
        env.host_string = 'otherhost'
        # env.host_string is now 'otherhost'
    # Outside the block:
    # * env.parallel is False again
    # * env.host_string remains 'otherhost'
    
#####Decorators#####

    fabric.decorators.hosts(\*host_list)  定义哪个或者哪些主机来执行这些命令,定义方式：
    @hosts('host1') 
    @hosts('host1', 'host2') 
    @hosts(['host1','host2']))
    fabric.decorators.roles(\*role_list) 定义执行任务的 roles,与主机对应
    env.roledefs.update({
        'webserver': ['www1', 'www2'],
        'dbserver': ['db1']})
    @roles('webserver', 'dbserver')
    def my_func():
        pass

同hosts一样，roles的参数既可以是一个参数列表，或者一个可迭代对象

    fabric.decorators.serial(func) （串行执行，不允许并行）强制func串行执行
    fabric.decorators.parallel(pool_size=None) （并行执行，而不是串行）   
    fabric.decorators.task(\*args, **kwargs) 即新风格任务，参照前面定义任务时的参数给其设定参数
    fabric.decorators.with_settings(\*arg_settings, **kw_settings) 作用同于上下文管理器中的with，只是它的作用域应该是整个函数。
    @with_settings(warn_only=True)
    def foo():
        ...
    fabric.decorators.runs_once(func) 函数仅执行一次 

#####Network#####
    
    fabric.network.disconnect_all()

用于与所有当前连接的服务器断开连接。一般用于 fab  主循环,也同时用于将Fabric作为类库使用。

#####Operations#####

    fabric.operations.get(remote_path, local_path=None)从 远 程 主 机下载一个或多个文件
    fabric.operations.put(local_path, remote_path, use_sudo=False, mirror_local_mode=False, mode=None) 从本地上传一个或多个文件至远程主机

很多时候用法类似scp或cp,用于上传下载文件，它的本地和远程路径实现发别是通过lcd和cd实现，所以用上下文管理器的cd会影响远程路径参数，而lcd会影响本地路径参数。这里主要是弄明白本地路径和远程路径的表示，它们将路径分成host/dirname/basename/path，可分别用基于字典的方式来指定，这里具体可查文档说明。

fabric.operations.open_shell(command=None) 在远程主机调用交互式shell。

如果命令给定，它将会在调用用户之前被送至管道。当你需要一个完全基于shell调用和一系列命令

时非常有用，比如debug的时候。这可以被看作是run的简单版，但并不是可替代版，因为它无法

处理远程交互中遇到的prompt，标准输入，login等问题。此外，它没有返回值而且在发生错误时也

不会有fabric的错误处理机制。

fabric.operations.reboot(wait=120) 重 启 远 程 机 器 , 会 暂 时 调 整fabric 冲连接设置以保证在 wait 时间内重连接成功。

fabric.operations.run(command, shell=True, pty=True, combine_stderr=True)在远程主机执行shell命令。

如果shell为True,run会将给定的命令通过env.shell设定的shell上解释。命令参数中任何双引号“和$符都会被忽略。

run会将远程程序的标准输出作为一个单一（可能多行）string作为结果返回。这个string包括布尔值的failed和succeed的属性来标志命令执行的成功与失败，

以及一个结果码来做为返回的 return_code属性。

本地终端的任何文本的输入在run执行过程中都会被送往远程程序，来允许你更自然地处理密码和其他prompts。

你可以使用pty=False来放弃在远程主机上创建伪终端以防其在命令存在问题时会制造麻烦。然而，这会强制fabric自己在run运行时echo所有的输入和你的键入，包括大小写敏感的密码。远程伪终端会替你echo，而且会只能的处理密码这种prompts。

同样地，如果你需要编程检测远程程序的标准错误流（利用函数返回值的stderr属性），你可以设置combine_stderr=False。这样做会使你终端输出很混乱（虽然返回strings能合理地分开）。但是这是你单独获取标准错误流的唯一方式。设置pty=combine_stderr=False。

fabric.operations.sudo(command, shell=True, pty=True, combine_stderr=True, user=None)

以上连个命令都是在远程执行命令,后者拥有特殊权限,关于参数含义已经在远程交互中详细介绍和run中介绍。

它们的返回结果是一个多行 string,这个 string 会有 failed, succeeded,return_code 这样的属性来标识是否执行成功。

同样地，如果你需要编程检测远程程序的标准错误流（利用函数返回值的stderr属性），你可以设置combine_stderr=False。这样做会使你终端输出很混乱（虽然返回strings能合理地分开）。

sudo会多接受一个user的参数，这用来允许你在其他用户而不是root用户来执行命令。在很多系统，user可以是string的username或者是int的uid。

此外上述各函数都有相应的参数为（*args, **kwargs）的重载函数。

fabric.operations.local(command, capture=False)

使用 python subprocess 模块实现且 shell=True,local 现在能够打印和捕捉输出,正如 run/sudo 一样。caputure 参数允许你选择是打印还是捕捉输出。

caputure=False 时,本地 subprocess 的标准输出和错误直接显示在终端,可通过全局输出控制,output.stdout 等,此时返回为空。

caputure=True 时,命令 stdout 作为类 string 对象返回,同 run/sudo 一样,返回值有 return_code,stderr,failed 和 succeeded 属性。


#####Tasks#####

class fabric.tasks.Task(alias=None, aliases=None, default=False, *args, **kwargs) 对于抽象基类和子类处理不同 详细参数含义可参考定义任务中新风格任务的参数含义。

fabric.tasks.execute(task, *args, **kwargs)  执行任务无论是新风格的任务名还是经典风格的可调用对象。

任务会在该任务主机列表中每个主机上执行一遍，这些主机列表包括来自命令行-H,装饰器@hosts,@roles,env.hosts等等。

host/hosts/role/roles、exclude_hosts都可以作为参数传递进去来设置该任务的主机列表。

如果task还有其他参数需要传递，比如

    execute(mytask, 'arg1', kwarg1='value')将会执行
    mytask('arg1', kwarg1='value')

它的执行结果也是按字典形式返回，比如execute(foo,hosts=['a', 'b'])

结果可能是{'a': None, 'b': 'bar'}

如果设置了skip_bad_hosts则该主机返回值可能为错误对象或消息。

#####Utils#####

fabric.utils.abort(msg)放弃执行，打印msg为stderr和错误状态。

fabric.utils.error(message, func=None, exception=None, stdout=None, stderr=None)调用func，给出指定错误、输出等信息。

fabric.utils.fastprint(text, show_prefix=False, end='', flush=True)

fabric.utils.indent(text, spaces=4, strip=False)同puts相同，只是不同的参数，快速打印文本text,不用等行结尾。

fabric.utils.puts(text, show_prefix=None, end='\n', flush=False)

fabric.utils.warn(msg)打印警告信息，但是不放弃执行。

***

####Contrib API####

一般使用较少，所以简单介绍

#####console output 工具#####

confirm(询问用户问题，返回 Y/N，比如“是否继续”这样的问题) 

#####django 集成#####

project:设置 DJANGO_SETTINGS_MODULE to '<name>.settings'. 

setting module：设置 DJANGO_SETTINGS_MODULE 

这个集成主要是用来做一些django设置方面的事，目前不成熟。

#####文件和目录管理#####

append(附加，类似 java 中 FileWriter 的 append 方法) 

contains(返回文件是否存在该文本) 

exits(返回远程主机是否存在该路径) 

first(返回多个文件中第一个找到的) 

upload_template(上传远程主机模板) 

#####项目工具#####

rsync_project(使用 rsync 同步远程目录与当前项目目录)

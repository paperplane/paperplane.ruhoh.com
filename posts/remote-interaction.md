---
title: Remote Interaction
date: '2013-03-17'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##远程交互--Remote Interaction##

Fabric基本操作，run和sudo，能够将本地输入发送到远程终端，以一种几乎和SSH相同的方式。这时你就像在和远程主机直接交互一样，例如程序提示输入密码之类。

然而，正如ssh本身一样，fabric在实现该功能时同样受到一些局限，因此并不总是很直观。

***

####合并stdout和stderr####

第一个需要讨论的问题就是标准输出和标准错误流，为什么它们能够根据需要分开和合并

Fabric 0.9以及之前版本，以至python自身，buffer输出都是基于line-by-line。除非一个新行字母（character）出现,否则文本不会被打印出来，这在大都数情况下工作的很好，但是当一个人需要处理半行输出(例如prompts)时会出现问题。

最新的fabric版本buffer输入和输出基于character-by-character，这使有prompts这样的交互变得可能。这对于那些复杂的需要利用‘curses’库或者需要重绘屏幕的程序(例如top这样)的交互提供了便利。

####交叉流####

不幸的是，同很多程序一样，同时打印stderr和stdout意味着两种流能同时独立地每次一byte打印，这会使结果混乱。虽然这个问题很多时候可以通过line-buffering其中某一种而不是另一种流来解决，这仍是一个问题。

为了解决这个问题，fabric使用一个SSH层的设置：它使两种流在一个低层次的级别合并然后输出，这让输出变得更自然。这是由env.combine_stderr呈现，默认为True。由于这种默认设置，输出变得正确，但是代价是sudo/run返回空的.stderr属性，因为所有输出都会呈现为stdout。

相反地，如果用户需要返回python级别的不同的stderr流而又不怕受到混乱的输出可以选择将其设置为False。因为combine_stderr=True时，即使发生错误，也只是标准输出，而不是标准错误。

####伪终端####

另一个需要考虑的问题就是什么时候激活prompt等待用户自己的输入

典型地终端应用程序或者文本终端呈献给程序以终端设备面孔，这称为tty或者pty(对于伪终端)。他们自动echo所有的文本输入然后将它们通过stdout返回给用户，因为交互时看不到已经输入的东西使它很困难。终端设备同样能够根据情况关闭echo,考虑到安全的密码prompts。

当然，程序也可以完全运行在没有tty或者pty的情况下（例如cron jobs），这时向这个程序所有输入数据都不会被echoed。这对于那些不需要人在旁边守候的程序很需要，同时它也是fabric之前默认的操作模式。

####Fabric的方法####

不幸的是，在使用fabric执行命令的上下文中，当没有pty提供给用户作为输入时，fabric必须为他们echo用户输入。这对于很多应用都没有问题，但是在密码输入时出现问题，它不再安全。

在满足安全和最少意外的双重要求下，fabric现在强制默认使用pty。当一个pty被启用，fabric可以简单地允许远程终端来处理echoing或者隐藏stdin并且不再echo任何东西。

需要关闭pty行为时，这时命令行选项--no-pty和env变量always_use_pty可能被用到。


####结合这两项####

作为最后的注释，记住有效使用伪终端意味着结合stdout和stderr。在很大程度和combine_stderr设置一样。这是因为一个终端设备自然向同一个地方---用户的显示屏，发送stdout和stderr，这使区分它们变得很难。

然而，在fabric级别，这两项可以组合成几种不同的设置。默认是两个都设置为True。默认设置在run方法中有详细介绍。此时既合并标准输出与标准错误，又在远程主机启动pty，不用Fabric自己echo所有输入。其他的组合如下。

run("cmd", pty=False, combine_stderr=True): 这会使fabric自己echo所有的输入，包括密码，以及其他潜在改变cmd的行为。一般是用在用户不关心密码提示以及在pty下cmd表现很糟的情况下。

run("cmd", pty=False, combine_stderr=False):这时fabric自己echo所有输入，而且也不会使用pty.这可能会导致除了最基础的命令外几乎所有的结果都很糟。然而，这是获取一个完全不同错误流的唯一方式。

run("cmd", pty=True, combine_stderr=False):有效，但是不会有什么作用，因为pty=True，就会合并流。此时等同于设置pty=combine_stderr=True。

默认方式，以及以上列举的第一种，第三种方式输出都是作为标准输出产生。使用默认方式会出项一个冗余信息就是正确执行时，返回结果stderr信息始终为空。

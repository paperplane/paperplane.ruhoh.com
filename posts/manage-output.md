---
title: Manage Output
date: '2013-03-16'
description:
categories: ['Fabric','OPS']
tags: ['Fabric']
---

##管理输出--Manage Output##

Fab工具默认输出很详细，包括：远端的stderr和stdout输出，被执行的命令内容等等。这在很多情况下能够让我们知道到底发生了什么。Fab工具输出最大的特点是分层次，根据需求来获取不同的部分。

***

####输出层次####

为了组织任务输出，Fabric输出被分成了不重叠的层次或者组，每一部分可以被单独开启或关闭。这为向用户提供输出提供了很多便捷性。

标准输出层次

status：状态消息，fabric运行时没有，如果用户使用keyboard interrupt，或者主机断开连接会输出相关消息。这些详细总是相关的，很少冗长。

aborts：放弃消息，像状态消息，这些消息应该只在fabric作为一个类库使用才关闭，因为即使你关闭了这个消息，在错误发生时，程序依然为停止，仅仅是不输出任何消息而已。

warning：警告消息，仅当期待某个操作时才关闭，比如说用grep去探测一个文件的存在性时，关闭它只会隐藏警告的消息，并不会控制警告出现这种行为。就如放弃消息一样。它的警告依然有只是不显示出来。

running：打印任务正在执行比如[myserver] run: ls /var/www

stdout：本地或者远程，标准输出

stderr：本地或者远程，标准错误

user：用户生成的输出

输出层次别名

    output：stdout and stderr
    command：stdout and running.
    everything：warnings, running, user and output

***

####显示/隐藏输出层次####

对于输出如果想控制输出层次，一般有两种方式：

(1)Context Manager：hide/show是一对context manager，它们采用一个或多个输出层次作为参数，将其中任意一个用在settings中即可。

with settings(show(‘stdout’,’stderr’))

在settings中可以嵌套调用hide/show

(2)命令行参数：fab  --show=LEVES  --hide=LEVES

注意多个输出层次中间用逗号（，）隔开，这与-H指定主机时一样,多个主机名用逗号隔开，但是作为参数传主机host string时又需要用分号


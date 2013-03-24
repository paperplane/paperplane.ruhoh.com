---
title: Puppet CA
date: '2013-03-24'
description:
categories: Puppet
tags: ['Puppet','DevOps','Python']

---

####概述####

puppet尽量依靠标准。就安全而言，它使用标准的SSL证书用于client和master的认证。这意味着客户端验证它与正确的server通信而且server验证它与正确的客户端通信。

由于为每个client颁发签名证书和管理自己认证权限复杂性的代价，puppet包括了它自己的认证授权（CA）。puppet并对使用这个认证授权进行了优化而且它也可被用于其他用途的生成证书。puppet证书管理主要目标是保持简单,并尽可能不让其更加明显。

puppetca是用于管理puppet认证授权的应用程序。它允许生成，撤销、签名、删除证书和显示签名请求列表。默认情况下，puppetmastered有认证授权中心功能。

<strong>证书</strong>

在puppetd或者puppetmasterde第一次执行时client和master自动生成证书，分别地，puppetd(client端puppet)，第一次连接master时会接受master证书并且保存。至此之后就会验证从master处获得证书的唯一性。

也可以通过手动将master证书通过安全通道复制到client端，需要注意中间人攻击。

***

<strong>客户端证书生成</strong>

没有签名证书的客户端会自动生成密钥对和证书请求，并在连接服务器端时一并提交给服务器端。如果服务器端开启自动认证，那么自动认证配置文件会检查客户端域名与其内容是否匹配。这个文件会在每次签名请求时被加载，所以任何更改会被很快应用。

<strong>服务器端证书管理</strong>

在通常情况下，证书的自动认证会被禁用。这样的情况下，证书必须通过使用puppetca工具进行签名。在1.0版之前，证书等待签名时会有邮件提醒，而现在可以通过日志或者puppetca --list来查看等待认证的请求列表。

一旦请求到达，使用puppetca --sign<hostname>来签名请求，增加--all标志会签名所有请求。所有已由puppet ca认证处理的证书列表在文件$cadir/inventory.txt中。

特定主机的所有认证文件可通过使用puppet ca --clean<hostname>来移除。这在某些情况下比如重装时会使用。
证书签名后可通过puppet revoke进行撤销，服务器端会在每次客户端建立连接时查看证书撤销列表（CRL）。对于每一次撤销的生效必须重启服务器端进程。

****

<strong>服务器端 客户端证书生成</strong>

通过使用puppetca -generate<hostname>可以在服务器端生成客户端证书，并且签名新生成的证书。这经常使用在自动化没有puppet管理的服务器端的puppet会话（应该是standalone安装方式），可以通过脚本将生成的公钥复制到客户端相应的位置，安装puppet并且执行puppetd来进行客户端配置。使用该命令会生成三个文件，$signeddir/hostname.pem, $certdir/hostname.pem and $privatekeydir/hostname.pem.需要将相应的文件拷贝到客户端相应的位置。


<strong>服务器端 客户端证书撤销</strong>

如果客客户端证书或者私钥受到破坏，可以撤销该客户端证书。

当客户端证书被撤销，它的序列号会被添加到当前的证书撤销列表（CRL），如果一个撤销的客户端连接puppetmaster来检查它的配置时会被拒绝连接，如果仅用于撤销客户端证书，CRL文件只需在puppetmaster上。

<strong>Python实现清除认证</strong>

下面实现使用web.py实现一个简单的清除认证接口

    import web
    import subprocess
    import logging
    import time
    import os

    urls = (
        '/', 'index',
        '/execution/([\w\W]*)', 'Exec',
    )

    class _AttributeString(str):
        @property
        def stdout(self):
            return str(self)

    class index:
        '''
        test page
        '''
        def GET(self):
            return "Hello, world!"

    class Exec:
        '''
        clean auth
        '''
        def POST(self,name):
        # command
        cmd = ['puppet','cert','clean']
        cmd.append(name)
        cmd = ' '.join(cmd)

        # exec
        p = subprocess.Popen(cmd,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
        (stdout,stderr) = p.communicate()
        out = _AttributeString(stdout)

        # result
        out.failed = False
        out.returncode = p.returncode
        out.err = stderr
        if out.returncode not in [0]:
            out.failed = True
        out.succeeded = not out.failed

        # logging
        logging.basicConfig(filename = os.path.join(os.getcwd(), 'log.txt'), level = logging.INFO)
        log = logging.getLogger('root')
        currtime = time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time()))
        if out.failed:
            log.error('['+currtime+'] Error!\nreturn code: ' + str(out.returncode) + '\n' + 'stderr: ' + out.err)
        else:
            log.info('['+currtime+'] Succeed!\n' + 'stdout: ' + out)

        # delete puppet reports and facts
        report_root = '/var/lib/puppet/reports/'
        facts_root = '/var/lib/puppet/yaml/facts/'
        facts_file = name + '.yaml'
        try:
            import shutil
            shutil.rmtree(os.path.join(report_root,name))
            os.remove(os.path.join(facts_root,facts_file))
        except (IOError, OSError):
            message = 'file or directory not exits'
            currtime = time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time()))
            log.error('['+currtime+'] Error!\n' + 'Error: ' + message)
            return out

    if __name__ == "__main__":
        app = web.application(urls, globals())
        app.run()

***

####认证过程举例####

客户端：

    $ puppetd --waitforcert 30 --server puppetserver.domain.net -v

服务器端：

    $ puppetca --list
    $ puppetca --sign puppetclient-37.domain.net


<strong>Puppet实现自动认证</strong>

1.puppetmaster给客户端签名,虽然方便了我们,但必须要注意安全,如果某个主机,正好请求到puppetmaster,你又自动签名,而它又执行了你默认的类,而类里有些秘密数据,那可就麻烦了.

2.实现puppet 客户端自动签名,需要两个步骤.

a. vim /etc/puppet/puppet.conf    2.6 版本为[master],2.7 版本为[main] 段,添加如下两行.

    autosign=true
    autosign = /etc/puppet/autosign.conf

b./etc/puppet/autosign.conf 
    
    *.test.com    # 域名
    192.168.1.0/24  #IP段

完成以上步骤即可实现自动给客户端ssl签名.

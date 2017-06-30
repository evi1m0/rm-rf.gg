---
layout: post
title: MIMTf [中间人攻击测试框架] - 入口源码分析
---
MITMf Framework 源码：[https://github.com/byt3bl33d3r/MITMf](https://github.com/byt3bl33d3r/MITMf)

上一篇介绍了 MITMf 这款中间人攻击测试框架，这里需要说一下的是，当我看完源码后才了解到 MIMTf 这款中间人攻击测试框架属于工具杂烩，然后使用框架进行统一管理并半规范化，所以本篇只对 mimtf.py 主入口源码进行简单分析。MIMTf 的框架目录结构：

    test@ff0000team:~/data/MITMf$ tree -L 2
    .
    ├── config
    │   ├── app_cache_poison_templates
    │   ├── mitmf.cfg
    │   └── responder
    ├── libs
    ├── LICENSE
    ├── lock.ico
    ├── logs
    │   ├── Analyze-LLMNR-NBT-NS.log
    │   ├── LLMNR-NBT-NS.log
    │   └── mitmf.log
    ├── mitmf.py
    ├── plugins
    ├── README.md
    ├── requirements.txt
    ├── setup.sh
    └── update.sh

简单分为主入口文件（mitmf.py）、库目录（libs）、攻击插件目录（plugins）、配置目录（config）：

- mitmf.py
- libs
- plugins
- config

#### MITMf.py Line 1-53

代码 1-20 行为声明版权不多讲，从 21 行开始加载模块：

    from twisted.web import http
    from twisted.internet import reactor

什么是 twisted ? (转自outofmemory)
 
> twisted是一个用python语言写的事件驱动的网络框架，他支持很多种协议，包括UDP,TCP,TLS和其他应用层协议，比如HTTP，SMTP，NNTM，IRC，XMPP/Jabber。非常好的一点是twisted实现和很多应用层的协议，开发人员可以直接只用这些协议的实现。其实要修改Twisted的SSH服务器端实现非常简单。很多时候，开发人员需要实现protocol类。
> 
> 一个Twisted程序由reactor发起的主循环和一些回调函数组成。当事件发生了，比如一个client连接到了server，这时候服务器端的事件会被触发执行。

    from libs.sslstrip.CookieCleaner import CookieCleaner
    from libs.sergioproxy.ProxyPlugins import ProxyPlugins
    from libs.banners import get_banner

加载 MITMf 封装模块，其中调用 [sslstrip](https://github.com/moxie0/sslstrip)、[sergio-proxy](https://github.com/supernothing/sergio-proxy) 及运行后提醒 banners 模块。

/libs/banners.py:

    def get_banner():
        banners = [banner1, banner2, banner3, banner4]
        return random.choice(banners)
        
定义多个 ASCII 图像然后随机选择返回，这里要吐槽的是 MITMf 框架代码中出现了相当多的冗余空格，例如行尾、空行等。

    import logging

    logging.getLogger("scapy.runtime").setLevel(logging.ERROR)  #Gets rid of IPV6 Error when importing scapy
    from scapy.all import get_if_addr, get_if_hwaddr

    from configobj import ConfigObj

    from plugins import *
    plugin_classes = plugin.Plugin.__subclasses__()

    import sys
    import argparse
    import os
    
导入 logging 日志模块、scapy、configobj、攻击插件等，其中 scapy 提一下：scapy是python写的一个功能强大的交互式数据包处理程序,可用来发送、嗅探、解析和伪造网络数据包,常常被用到网络攻击和测试中。

    try:
        import netfilterqueue
        if netfilterqueue.VERSION[1] is not 6:
            print "[-] Wrong version of NetfilterQueue library installed!" 
            print "[-] Download it from here https://github.com/fqrouter/python-netfilterqueue and manually install it!"
    except ImportError:
        print "[-] NetfilterQueue library missing! DNS tampering will not work"

    try:
        import user_agents
    except ImportError:
        print "[-] user_agents library missing! User-Agent parsing will be disabled!"
        
[netfilterqueue](https://pypi.python.org/pypi/NetfilterQueue/0.3)：NetfilterQueue provides access to packets matched by an iptables rule in Linux. Packets so matched can be accepted, dropped, altered, or given a mark.

版本或导入异常会在运行时提示 WARNING 信息，后面则导入 user_agents，同理。

#### MITMf.py Line 55-80

定义 MITMf 常量，输出 banner ASCII 图像效果，定义工具参数：

    mitmf_version = "0.9.5"
    sslstrip_version = "0.9"
    sergio_version = "0.2.1"

    banner = get_banner()
    print banner

    parser = argparse.ArgumentParser(description="MITMf v%s - Framework for MITM attacks" % mitmf_version, version=mitmf_version, usage='', epilog="Use wisely, young Padawan.",fromfile_prefix_chars='@')
    #add MITMf options
    mgroup = parser.add_argument_group("MITMf", "Options for MITMf")
    mgroup.add_argument("--log-level", type=str,choices=['debug', 'info'], default="info", help="Specify a log level [default: info]")
    ...........
    #add sslstrip options
    sgroup = parser.add_argument_group("SSLstrip", "Options for SSLstrip library")
    slogopts = sgroup.add_mutually_exclusive_group()
    slogopts.add_argument("-p", "--post", action="store_true",help="Log only SSL POSTs. (default)")
    ...........
    
#### MITMf.py Line 80-117

初始化插件表，遍历 plugin_classes 添加进入 plugins 列表中：

    plugins = []
    try:
        for p in plugin_classes:
            plugins.append(p())
    except:
        print "Failed to load plugin class %s" % str(p)

给予每个插件对应的选项及描述：

    try:
        for p in plugins:
            if p.desc == "":
                sgroup = parser.add_argument_group("%s" % p.name,"Options for %s." % p.name)
            else:
                sgroup = parser.add_argument_group("%s" % p.name, p.desc)

            sgroup.add_argument("--%s" % p.optname, action="store_true",help="Load plugin %s" % p.name)

            if p.has_opts:
                p.add_options(sgroup)
    except NotImplementedError:
        sys.exit("[-] %s plugin claimed option support, but didn't have it." % p.name)

将参数赋值给 args 后，ConfigObj 导入 mitmf 配置：

> args.configfile：
> 
> mgroup.add_argument("-c", "--config-file", dest='configfile', type=str, default="./config/mitmf.cfg", metavar='configfile', help="Specify config file to use")

    args = parser.parse_args()

    try:
        configfile = ConfigObj(args.configfile)
    except Exception, e:
        sys.exit("[-] Error parsing config file: " + str(e))
        
#### MITMf.py Line 118-158

检查加载的插件是否需要 ROOT 权限执行：

    try:
        for p in plugins:
            if (vars(args)[p.optname] is True) and (p.req_root is True):
               if os.geteuid() != 0:
                    sys.exit("[-] %s plugin requires root privileges" % p.name)
    except AttributeError:
        sys.exit("[-] %s plugin is missing the req_root attribute" % p.name)

一些比较有意义的变量检查，例如网卡信息、MAC地址等：
        
    try:
        args.ip_address = get_if_addr(args.interface)
        if (args.ip_address == "0.0.0.0") or (args.ip_address is None):
            sys.exit("[-] Interface %s does not have an assigned IP address" % args.interface)
    except Exception, e:
        sys.exit("[-] Error retrieving interface IP address: %s" % e)

    try:
        args.mac_address = get_if_hwaddr(args.interface)
    except Exception, e:
        sys.exit("[-] Error retrieving interface MAC address: %s" % e)
        
日志信息记录：

    logging.basicConfig(level=log_level, format="%(asctime)s %(message)s", datefmt="%Y-%m-%d %H:%M:%S")
    logFormatter = logging.Formatter("%(asctime)s %(message)s", datefmt="%Y-%m-%d %H:%M:%S")
    rootLogger = logging.getLogger()

    fileHandler = logging.FileHandler("./logs/mitmf.log")
    fileHandler.setFormatter(logFormatter)
    rootLogger.addHandler(fileHandler)
    
#### MITMf.py Line 159-174

开始加载所有的插件并打印相应插件信息：

    print "[*] MITMf v%s online... initializing plugins" % mitmf_version

    load = []

    for p in plugins:
        try:
            if vars(args)[p.optname] is True:
                print "|_ %s v%s" % (p.name, p.version)

            if getattr(args, p.optname):
                p.initialize(args)
                load.append(p)
        except Exception, e:
            print "[-] Error loading plugin %s: %s" % (p.name, str(e)) 
            
#### MITMf.py Line 175-207

判断使用者是否开启 -d 参数来禁用所有代理，只运行插件：

    if args.disproxy:
        ProxyPlugins.getInstance().setPlugins(load)
    else:
   		...

现在所有准备都已经完成，MITMf 正式工作了：

    from libs.sslstrip.StrippingProxy import StrippingProxy
    from libs.sslstrip.URLMonitor import URLMonitor

    URLMonitor.getInstance().setFaviconSpoofing(args.favicon)
    CookieCleaner.getInstance().setEnabled(args.killsessions)
    ProxyPlugins.getInstance().setPlugins(load)

    strippingFactory              = http.HTTPFactory(timeout=10)
    strippingFactory.protocol     = StrippingProxy

加载自定义的插件选项，配置 twisted listenTCP ，listen 默认值为 10000：

    reactor.listenTCP(args.listen, strippingFactory)
    
    for p in plugins:
        if getattr(args, p.optname):
            if hasattr(p, 'plugin_reactor'):
                p.plugin_reactor(strippingFactory) #we pass the default strippingFactory, so the plugins can use it

树状打印信息，表示正在运行中：

    print "|"
    print "|_ Sergio-Proxy v%s online" % sergio_version
    print "|_ SSLstrip v%s by Moxie Marlinspike running...\n" % sslstrip_version

这两行代码至关重要，它会启动reator的主循环：

    from twisted.internet import reactor
    reactor.run()

退出运行每个插件 finish：

    for p in load:
        p.finish()
        
> 转载请著名出处，作者：Evi1m0#linux.im
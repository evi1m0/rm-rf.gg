---
layout: post
title: MIMTf [中间人攻击测试框架] - Introduction
---

#### MIMTf 介绍

- Github 项目主页：[https://github.com/byt3bl33d3r/MITMf](https://github.com/byt3bl33d3r/MITMf)
- 开发案例及更新  ：[http://sign0f4.blogspot.it/](http://sign0f4.blogspot.it/)
- MIMTf  项目作者：[byt3bl33d3r](https://github.com/byt3bl33d3r)

MIMTf 全称为 Framework for Man-In-The-Middle attacks （中间人攻击框架），它基于 sergio-proxy 代理工具修改而来，由 Python 强势驱动。#### MIMTf 可用插件

* Responder - LLMNR, NBT-NS and MDNS poisoner
* SSLstrip+ - Partially bypass HSTS
* Spoof - Redirect traffic using ARP Spoofing, ICMP Redirects or DHCP Spoofing and modify DNS queries
* Sniffer - Sniffs for various protocol login and auth attempts
* BeEFAutorun - Autoruns BeEF modules based on clients OS or browser type
* AppCachePoison - Perform app cache poison attacks
* SessionHijacking - Performs session hijacking attacks, and stores cookies in a firefox profile
* BrowserProfiler - Attempts to enumerate all browser plugins of connected clients
* CacheKill - Kills page caching by modifying headers
* FilePwn - Backdoor executables being sent over http using bdfactory
* Inject - Inject arbitrary content into HTML content
* JavaPwn - Performs drive-by attacks on clients with out-of-date java browser plugins
* jskeylogger - Injects a javascript keylogger into clients webpages
* Replace - Replace arbitary content in HTML content
* SMBAuth - Evoke SMB challenge-response auth attempts
* Upsidedownternet - Flips images 180 degrees

#### MIMTf 安装

如何安装在 kali 系统上:

    apt-get install mitmf

下面为我在 ubuntu 12.04 LTS 下的安装：

    test@ff0000team:~/data$ git clone https://github.com/byt3bl33d3r/MITMf.git
    test@ff0000team:~/data$ cd MITMf/
    test@ff0000team:~/data/MITMf$ sudo ./setup.sh
    
查看 requirements.txt 可以看到 MIMTf 的依赖较多：

    Twisted
    requests
    scapy
    msgpack-python
    dnspython
    user-agents
    configobj
    pyyaml
    NetfilterQueue >= 0.6
    ua-parser
    Pillow
    pefile
    capstone
    
安装完成后运行：

    python mitmf.py --help
    
不出意外的话，会爆缺少模块依赖的错误，使用 pip 或者 easy_install 进行安装即可，另外需要提一点因为依赖库中有谷歌源，所以如果未翻墙的话可能会下载依赖失败，例如：pefile ，可自行去 pypi 官方进行下载解压安装。

有时包名与安装名不同，例如：

    ImportError: No module named pcap
    sudo apt-get install python-libpcap

    ImportError: No module named msgpack
    easy_install msgpack-python

另外项目主页 README 中有关 Dependency change! 的提醒：

> As of v0.9.5 DNS tampering support needs NetfilterQueue v0.6 which is currently a fork, so it cannot be installed via pip or easy_install.

是有关 netfilterqueue 的依赖问题，解决方案：

    ImportError: No module named netfilterqueue
    apt-get install build-essential python-dev libnetfilter-queue-dev
    pip install NetfilterQueue
    
或直接访问 netfilterqueue 项目主页下载安装：

    Github: https://github.com/kti/python-netfilterqueue
    
#### MITMf 运行

    test@ff0000team:~/data/MITMf$ python mitmf.py --help

    ----------- BANNER ----------    

    usage: 

    MITMf v0.9.5 - Framework for MITM attacks

    optional arguments:
      -h, --help            show this help message and exit
      -v, --version         show program's version number and exit

如何判断是否正常运转？

运行 mitmf.py 入口文件后， BANNER (LOGO) 上方未出现下面等错误信息字样。

	[-] Wrong version of NetfilterQueue library installed!
	[-] Download it from here https://github.com/fqrouter/python-netfilterqueue and manually install it!
    [-] user_agents library missing! User-Agent parsing will be disabled!至此 MITMf 正常运行，安装完毕。
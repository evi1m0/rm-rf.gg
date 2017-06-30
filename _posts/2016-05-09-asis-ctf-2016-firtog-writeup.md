---
layout: post
title: Asis CTF 2016 - Firtog WriteUP
---
#### Forensic / firtog 109

> [Obscurity](https://asis-ctf.ir/tasks/firtog_75813ce76bbf9014bb7afae8071e180e0b939d31713062baf3ffafd852f1f7687e8d1cf61762bb80245adf3fec5cbcf01e3e746bbaf3a880fd13f61b122080c4) is definitely not security.

周末和朋友打完 DotA2 发现 Asis2016 开始了，和死猫随便挑了道取证题来做练练脑力，后来我睡着了直到第二天中午醒来他告诉我已经搞定了这道题。随后自己用了另外一个想法搞定了这道题，可能和死猫的思路不太一样，简单记录一下。

    ➜  testfirtog file firtog_75813ce76bbf9014bb7afa
    firtog_75813ce76bbf9014bb7afa: xz compressed data
    
    ➜  testfirtog unxz _firtog.xz
    
    ➜  testfirtog file _firtog
    _firtog: POSIX tar archive
    
    ➜  testfirtog tar -xvf _firtog
    x firtog.pcap

这里我们获取到 pcap 数据包文件，先用 strings 扫一眼看看大概情况：

![https://ooo.0o0.ooo/2016/05/09/5730a51318de4.png](https://ooo.0o0.ooo/2016/05/09/5730a51318de4.png)

看样子是 git 操作的流量包，在包内容中发现了一些值得引起注意的字符串，如：

- 100755 flag.py
- 100644 readme
- 0037git-receive-pack /asisPrivRepo
- Compressing objects: 100% (3/3), done.
- ...

这时我认为题目大概考的就是还原出这个 git 项目运行相应的脚本来获取 flag ，现在使用 Wireshark 将 pcap 文件导入浏览，获取 TCP 流内容：

![https://ooo.0o0.ooo/2016/05/09/5730a68927dc5.png](https://ooo.0o0.ooo/2016/05/09/5730a68927dc5.png)

翻阅资料找到了 git pack-protocol ，但是没有成功的将流量包还原成正确 packfile 。如果能够还原 packfile 的话，则可以使用 git [unpack-objects](https://git-scm.com/docs/git-unpack-objects) <  packfile 的方法来打开对象包然后恢复结构。

git 包协议的压缩方法是用的 zlib ，这里简单提一下由于 zlib 压缩数据是以 x78 字节开始的：

![https://ooo.0o0.ooo/2016/05/09/5730a956155b6.png](https://ooo.0o0.ooo/2016/05/09/5730a956155b6.png)

所以我们可以编写一个脚本来遍历寻找数据并解码输出：

    ➜  testfirtog cat test.py
    #!/usr/bin/env python
    # asis ctf firtog
    # author: evi1m0
    
    outfile  = open('/tmp/firtog', 'a+')
    pcapfile = open('firtog.pcap').read()
    
    while 1:
        v = pcapfile.find('\x78')
        if v:
            try:
                outfile.write(pcapfile[v:].decode('zlib'))
            except:
                pass
            pcapfile = pcapfile[v + 2:]

![https://ooo.0o0.ooo/2016/05/09/5730aa25ce64e.jpg](https://ooo.0o0.ooo/2016/05/09/5730aa25ce64e.jpg)

拿到解除的数据之后，其中相关文件源码如下：

    flag.py
        # Simple but secure flag generator for ASIS CTF
    
        from os import urandom
        from hashlib import md5
    
        l = 128
        rd = urandom(l)
        h = md5(rd).hexdigest()
        flag = 'ASIS{' + h + '}'
        f = open('flag.txt', 'r').read()
        flag = ''
        for c in f:
            flag += hex(pow(ord(c), 65537, 143))[2:]
        print flag
    
    flag.txt
        41608a606a63201245f1020d205f1612147463d85d125c1416635c854c74d172010105c14f8555d125c3c

然后我们编写对应的恢复脚本来跑这个 flag.txt 就能解出这个 key 究竟是什么了：

![https://ooo.0o0.ooo/2016/05/09/5730ab126848e.jpg](https://ooo.0o0.ooo/2016/05/09/5730ab126848e.jpg)

这道题还是蛮有趣的，只是郁闷的地方在于开始不会使用 git unpack 来正常操作流程，后来在国外的 WriteUP 上看到了 unpack 正规的解题思路（链接最后给出），但我仍然觉得这个野路子挺好的。 xD

#### 参考

- [https://git-scm.com/docs/git-unpack-objects](https://git-scm.com/docs/git-unpack-objects)
- [https://systemoverlord.com/2016/05/08/asis-ctf-2016-firtog.html](https://systemoverlord.com/2016/05/08/asis-ctf-2016-firtog.html)
- [https://github.com/git/git/blob/master/Documentation/technical/pack-protocol.txt](https://github.com/git/git/blob/master/Documentation/technical/pack-protocol.txt)
- [http://book.51cto.com/art/200903/112931.htm](http://book.51cto.com/art/200903/112931.htm)
- [http://blog.jobbole.com/26209/](http://blog.jobbole.com/26209/)
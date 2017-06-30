---
layout: post
title: Docker install Kali Linux
---
#### 什么是 Kali linux

Kali Linux 是基于 Debian 的 Linux 发行版， 设计用于数字取证和渗透测试。由 Offensive Security Ltd 维护和资助。最先由 Offensive Security 的 Mati Aharoni和Devon Kearns 通过重写 BackTrack 来完成，BackTrack 是他们之前写的用于取证的Linux发行版。

Kali Linux预装了许多渗透测试软件，包括nmap (端口扫描器)、Wireshark (数据包分析器)、John the Ripper (密码破解器),以及Aircrack-ng (一应用于对无线局域网进行渗透测试的软件).用户可通过硬盘、live CD 或 live USB 运行Kali Linux。Metasploit的 Metasploit Framework 支持 Kali Linux，Metasploit 一套针对远程主机进行开发和执行 Exploit 代码的工具。

#### 什么是 Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架包括系统。

#### 为什么要使用 Docker

作为一种新兴的虚拟化方式，Docker 跟传统的虚拟化方式相比具有众多的优势。

首先，Docker 容器的启动可以在秒级实现，这相比传统的虚拟机方式要快得多。 其次，Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器。

容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而Docker 只需要启动 10 个隔离的应用即可。

具体说来，Docker 在如下几个方面具有较大的优势：

- 更快速的交付和部署
- 更高效的虚拟化
- 更轻松的迁移和扩展
- 更简单的管理

#### Mac OS 安装 Docker

由于 Docker 暂时并不支持原生的 Mac 系统，所以 Mac 下的 Docker 实际上是依赖一个很小的linux虚拟机来实现的。Vagrant依赖现有的虚拟机软件来管理虚拟机，如Virtualbox, Vmware Fusion, Parallel Desktop等。

Boot2Docker 是帮助控制虚拟机中 Docker 的工具，它会下载一个安装好docker的虚拟机，并控制其实现docker功能：

	brew install boot2docker

安装 Docker client：

    # Get the docker client file
    DIR=$(mktemp -d ${TMPDIR:-/tmp}/dockerdl.XXXXXXX) && \
    curl -f -o $DIR/ld.tgz https://get.docker.io/builds/Darwin/x86_64/docker-latest.tgz && \
    gunzip $DIR/ld.tgz && \
    tar xvf $DIR/ld.tar -C $DIR/ && \
    cp $DIR/usr/local/bin/docker ./docker

    # Set the environment variable for the docker daemon
    export DOCKER_HOST=tcp://127.0.0.1:4243

    # Copy the executable file
    sudo cp docker /usr/local/bin/#### Kali-Linux-Docker

Image: [https://registry.hub.docker.com/u/kalilinux/kali-linux-docker/](https://registry.hub.docker.com/u/kalilinux/kali-linux-docker/)

> Official Kali Linux Docker
>
> This Kali Linux Docker image provides a minimal base install of the latest version of Kali Linux 1.x. There are no tools added to this image, so you will need to install them yourself. For details about Kali Linux metapackages, check https://www.kali.org/news/kali-linux-metapackages/

简介中说到：镜像提供了一个纯净的 Kali-linux 最新版，没有任何工具添加在这个镜像包中，所以需要使用者自己安装他们。

#### Install kali-linux-docker

    boot2docker start
    Waiting for VM and Docker daemon to start...
    .........ooo
    Started.
    Writing /Users/evi1m0/.boot2docker/certs/boot2docker-vm/ca.pem
    Writing /Users/evi1m0/.boot2docker/certs/boot2docker-vm/cert.pem
    Writing /Users/evi1m0/.boot2docker/certs/boot2docker-vm/key.pem
    Your environment variables are already set correctly.
    
    docker pull kalilinux/kali-linux-docker
    
启动完毕 Docker 运行 docker pull repo ，这时发生了一个错误：

    FATA[0000] Post http:///var/run/docker.sock/v1.18/images/create?fromImage=kalilinux%2Fkali-linux-docker%3Alatest: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?
    
解决方法如下，可以考虑把它写入 ~/.zshrc 或者 ~/.bash_profile 中：

    $(boot2docker shellinit 2> /dev/null)
    
安装并运行：

    ➜  ~  docker pull kalilinux/kali-linux-docker
    latest: Pulling from kalilinux/kali-linux-docker

    a4d244f4db27: Pull complete
    de27dae99aeb: Pull complete
    f90369c38e73: Pull complete
    3c8200eafd33: Pull complete
    cf17f2bea29d: Pull complete
    7357a6752463: Pull complete
    55f49aa24e98: Already exists
    Digest: sha256:b827275b6de61135767c8928c9f0dbd620a8f7f782f7e85c02d5323255a212fb
    Status: Downloaded newer image for kalilinux/kali-linux-docker:latest
    ➜  ~  docker images
    REPOSITORY                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    kalilinux/kali-linux-docker   latest              55f49aa24e98        4 days ago          354.7 MB
    ubuntu                        12.04               9c5e4be642b7        9 weeks ago         131.9 MB
    ➜  ~  docker run -t -i kalilinux/kali-linux-docker /bin/bash
    root@73e6c7edf10c:/# ls
    bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
    
其中，-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。

#### 参考文献

- [http://stackoverflow.com/questions/27528337/am-i-trying-to-connect-to-a-tls-enabled-daemon-without-tls](http://stackoverflow.com/questions/27528337/am-i-trying-to-connect-to-a-tls-enabled-daemon-without-tls)
- [http://dockerpool.com/static/books/docker_practice/index.html](http://dockerpool.com/static/books/docker_practice/index.html)
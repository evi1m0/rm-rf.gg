---
layout: post
title: BPG Image format
---

#### 简介

[BPG](http://bellard.org/) (Better Portable Graphics)是一个新的图像格式，从全称不难看出他的优点。

他们诞生其目的是为了取代JPEG图像格式，但图像质量和大小一直都是个问题。

#### 优点

- 高度压缩比，官方称比相同类型图片的JPEG图像要小的多
- 大部分浏览器都支持，不支持可安装JavaScript解码器
- 支持无损压缩
- 支持传统的元数据

#### DEMO

- [BPG examples](http://bellard.org/bpg/gallery1.html)
- [BPG/JPEG](http://bellard.org/bpg/lena.html)
- [BPG/PNG](http://bellard.org/bpg/gallery2.html)

#### 更多的信息

- [http://bellard.org/bpg/bpg_spec.txt](http://bellard.org/bpg/bpg_spec.txt)#### 最后

按DEMO来看压缩比确实有到不错的提升，但影响其发展的因素依然有很多。

#### 2014-12-1:更新

xooyoozoo编写了一个BPG图像的比较工具很有意思，源码：[https://github.com/xooyoozoo/yolo-octo-bugfixes/tree/gh-pages](https://github.com/xooyoozoo/yolo-octo-bugfixes/tree/gh-pages)

在线使用地址：[http://xooyoozoo.github.io/yolo-octo-bugfixes/#soccer-players&webp=s&bpg=l](http://xooyoozoo.github.io/yolo-octo-bugfixes/#soccer-players&webp=s&bpg=l)
---
layout: post
title: Bypass NoScript Vulnerability ( &lt; 2.6.9.28rc1 ) 
---
- Discovery Date: 30-06-15
- Affected Products: NoScript Plugin < 2.6.9.28rc1

#### About this Add-on

这款扩展曾经获得 “2006 PC World World Class Award” 大奖。

NoScript 可以根据用户的选择，只允许受信任的网站启用 JavaScript、Java 或其他插件。白名单基于优先阻止机制，在不损失任何功能的情况下，阻止利用已知或未知安全漏洞的攻击。

它只允许在您选择的信任域（例如您的家庭银行网站）上执行JavaScript、Java和其他插件，保护您的”信任边界“不受跨站的XSS攻击、跨区的DNS重新绑定/CSRF攻击（路由器黑客）和点击劫持尝试，这些全都多亏了它独特的ClearClick技术。它还实现了默认使用请勿跟踪选择退出建议。

这种抢先阻止机制在不损失任何功能的前提下，防止利用已知的或未知的安全漏洞的攻击。

专家们一致同意：拥有 NoScript 的 Firefox 更安全 :-)

#### Vulnerability

在 NoScript 代码中有一段标注白名单信任域：*.googleapis.com ，导致我们可以使用 Google Cloud (storage.google.com) 提供的服务绕过检测机制，执行恶意代码。

又是一款典型信任过度导致的安全问题：

Demo: [https://storage.googleapis.com/zulln/alert.htm](https://storage.googleapis.com/zulln/alert.htm)

#### Fix

- 升级官方最新版本

刚才去官方拿源码进行了对比，现在已经将 googleapis.com 修改为了 ajax.googleapis.com :-)

![http://ww2.sinaimg.cn/large/c334041bgw1etn3qmj9rfj20lf08mtd2.jpg](http://ww2.sinaimg.cn/large/c334041bgw1etn3qmj9rfj20lf08mtd2.jpg)
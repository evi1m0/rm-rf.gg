---
layout: post
title: 巧遇 Dota2 钓鱼，我的做法是？
---
DOTA2 是脱离了其上一代作品《DOTA》所依赖的War3的引擎，由《DOTA》的地图核心制作者IceFrog（冰蛙）联手美国Valve公司使用他们的Source引擎研发的、Valve运营，完美世界代理（国服），韩国NEXON代理（韩服）的多人联机对抗RPG。DOTA2 游戏保持原有风格不变，《DOTA》中的100多位英雄正在逐步的移植到《DOTA2》中。

Dota2 拥有大量的游戏玩家，最近我被同事拉着玩了一段时间，确实很有趣，因工作时间原因导致我养成了收集饰品的习惯，喜欢睡前刷刷饰品买些古董。在这个前提下，我有试着了解 Dota2 民间市场以及交易的一些安全机制，仅仅是发现了一些小打小闹的漏洞，至于 Dota2 钓鱼，之前还真没听说过，有趣的是今天下班时我终于遇到了。“叮”，Steam 上面传来好友添加的审核提示，添加成功后对面发来这样一条消息：

![http://ww3.sinaimg.cn/large/c334041bgw1etvj8ckofmj20m00jiq57.jpg](http://ww3.sinaimg.cn/large/c334041bgw1etvj8ckofmj20m00jiq57.jpg)

哦，应该是 Dota2 市场想与我交换饰品的朋友，注意这段话有几点：想买你的东西、好的价钱、加我好友和一条链接，说明缘由 -> 诱使动机 -> 动作指引 -> 目标链接。

按照对方的指引我们进入 Dota2 市场交易页面，这时的操作无论是添加他的好友还是查看他的库存饰品，都需要点击网页中的昵称，进入对方主页：

![http://ww2.sinaimg.cn/large/c334041bgw1etvj8r04voj20jv0ch0uz.jpg](http://ww2.sinaimg.cn/large/c334041bgw1etvj8r04voj20jv0ch0uz.jpg)

在点击查看对方主页后，通常会跳转到登录  Steam 帐号的页面（强制设定环境为未登录状态），从跳转前到跳转后的页面一直都是假的，因为域名仅是多了个 .cn ，导致不少人是认为没有问题的，这个钓鱼点就是用户 Steam 帐号密码。注：在 Steam 当中异地登录是需要目标邮箱接收验证码的。

在随后的过程中，我拿下这个钓鱼网站发现已经有 6,000 余人的帐号密码被贴在上面：

![http://ww4.sinaimg.cn/large/c334041bgw1etvj9c2zqbj20ed0dr791.jpg](http://ww4.sinaimg.cn/large/c334041bgw1etvj9c2zqbj20ed0dr791.jpg)

我们来试一下登录其中一个 Steam ：

![http://ww2.sinaimg.cn/large/c334041bgw1etvj9vi5btj20ak0boac1.jpg](http://ww2.sinaimg.cn/large/c334041bgw1etvj9vi5btj20ak0boac1.jpg)

异地登录拦截提示给我们的信息有两点：你要告诉我验证码，你的邮箱服务商，这使得能让我们轻松的利用被钓鱼的帐号密码配合 Steam 提供给的邮箱提供商成功的登录邮箱接收验证码：

![http://ww4.sinaimg.cn/large/c334041bgw1etvjayybjwj20by05uq3a.jpg](http://ww4.sinaimg.cn/large/c334041bgw1etvjayybjwj20by05uq3a.jpg)

让我来揣测下整个过程：

- 爬虫获取交易额度较高的用户 Steam ID；
- 批量添加这些好友并发送添加成功后的消息；
- 提供一个用于钓鱼的网站；
- 等待收鱼后，批量的进行登录测试，留下正确的帐号密码；
- 洗号阶段 — 配合 Steam 提供的邮箱服务商信息，进行社工盗取游戏内物品；

嗯，这下你应该不难理解：为什么黑产总是走在前沿了吧 ：）
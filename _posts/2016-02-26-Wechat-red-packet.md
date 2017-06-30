---
layout: post
title: 通用型微信抢红包神器 - 已修复
---
本篇文章记录 2015年初时利用微信网页版红包 H5 接口实现的通用型平台抢红包神器，不同于目前市面上的抢红包外挂（模拟点击、HookAPI等）方式，当然该漏洞在15年3月便已经修复。

#### WebAPI

我们在使用网页版微信时无法点击领取红包，但是通过控制台查看网络包发现红包地址在 Response 中：

    https://wxapp.tenpay.com/app/v1.0/receive.cgi?showwxpaytitle=1&sendid=1000033901********10153974107&channelid=1&msgtype=1&ver=2&sign=aefc0f0fadb6f09993fb071bf3****************9b4d8020f07c351c4bb6489d60d65f7f9fb2622f977fa613cbe443f6c7279f03fff2aff18859674c978656ed45388e3f606eae42a0f8ef9014f1***********21e055163c935

![http://ww1.sinaimg.cn/large/c334041bjw1f1cpjydvuuj20nv05fab1.jpg](http://ww1.sinaimg.cn/large/c334041bjw1f1cpjydvuuj20nv05fab1.jpg)

当群组或者用户给我们发红包时可以在返回内容中可以看到红包的 URL 地址（手机端老版本的微信是使用 H5 的接口领取红包，而无法自动判断用户发来的消息是否为红包，网页版本微信解决了判断是否为红包消息的问题，并且能获取 URL）。

直接在浏览器访问获取到的红包 URL，提示来源错误。任何 HTTP 请求重放都是简单的，之后使用 Mitmproxy 获取手机抢红包时的 Cookie,User-Agent 等 Header 头，然后进行重放包是可以成功领取红包的。

![http://ww2.sinaimg.cn/large/c334041bjw1f1cpqmffzyj20o109oab8.jpg](http://ww2.sinaimg.cn/large/c334041bjw1f1cpqmffzyj20o109oab8.jpg)

#### Process

流水线：

- 使用 JavaScript 对微信网页版 Hook 实时监听，判断收到消息是否为红包，如果是则匹配；
- 使用 Mitmproxy 对手机端进行抓包，获取 Header 信息；
- 利用获取到的头信息填充至开始获得到的红包地址，领取红包；

自动化：

- 编写填充头信息以方便打开红包的 Python 脚本；
- 编写实时监控浏览器中网络传输红包数据的 JavaScript 脚本；
- 考虑方便本机搭建 Mini Django 项目，在获取到红包 URL 后自动发送请求给后端抢红包[1]；
- ...

效果图：

![http://ww1.sinaimg.cn/large/c334041bjw1f1cq1mv0jtj207s04imxg.jpg](http://ww1.sinaimg.cn/large/c334041bjw1f1cq1mv0jtj207s04imxg.jpg)

#### Code

wechat_web_hook.js:

    /*
     * HookJS @ evi1m0, fyth, erevus.
     * Datetime @ 2015
     */

    $._ajax = $.ajax;

    function check(c){
        var AddMsgList = c.AddMsgList;
        if (AddMsgList) {
            AddMsgList.forEach(function(elem, index){
                var str = '';
                str = elem.Url || '';
                var bingo = /https:\/\/wxapp.tenpay.com/.test(str);
                if(bingo) {
                    str = str.replace(/amp;/g, '')
                    console.log(str);
                    $.post('http://localhost:1234/wechat', {url: str})
                }
            });
        }
    }

    function a(e, n){
        e._s = e.success;
        e.success = function(data) {
            try{
                check(data, this);
                this._s.apply(this, arguments);
            }
            catch(e){

            }
        }
        c = $._ajax(e, n);
        return c;
    }
    $.ajax = a;
    
views.py
    
    #!/usr/bin/env python
    # author  : evi1m0, rains
    # datetime: 201502

    from django.shortcuts import render
    from django.http import HttpResponse

    import re
    import sys
    import json
    import time
    import base64
    import urllib
    import urllib2
    import requests

    def index(request):
        url = 'http://www.baidu.com'
        req = urllib2.urlopen(url)
        content = req.read()
        return HttpResponse(content)    def open(request, url):
        url = urllib.unquote(url).replace('amp;', '')
        print url
        cookie = "test NULL"
        return HttpResponse(_money(url, cookie))    def _money(url, cookie):
        headers_fake = {'User-Agent': ("Mozilla/5.0 (Linux; U; Android 4.2.2; zh-cn; Galaxy Nexus - 4.2.2 - with Google Apps"
                                       "- API 17 - 720x1280 1 Build/JDQ39E) AppleWebKit/534.30 (KHTML, like Gecko)"
                                       "Version/4.0 Mobile Safari/534.30 MicroMessenger/5.3.0.80_r701542.440"),
                        'X-Requested-With': 'com.tencent.mm',
                        'Cookie': cookie,
                       }

        try:
            req_key_content = requests.get(url, headers=headers_fake).content
        except Exception,e:
            print e

        try:
            g_sendld = re.findall('g_sendId: "(.*?)",', req_key_content)
            g_sendnick = re.findall('g_sendNick: "(.*?)",', req_key_content)
            g_detailtoke = re.findall('g_detailToke: "(.*?)",', req_key_content)
            g_detailtoke = (g_detailtoke[0])
        except Exception, e:
            print '[-] Error: %s' % str(e)
            return False

        sign_key = (str(url.split('&')[-1:]).split('=')[1])[:-2]
        receive_url = ("https://wxapp.tenpay.com/app/v1.0/wxhb_open.cgi?msg_type=&send_id=%s&channel_id=&detailToke=%s&scene="
                       "&us=&sign=%s&attention=0&hb_version=v2&ver=2&isappinstalled=0&clientversion=25030050&devicetype=android"
                       "-17")
        receive_url = receive_url % (g_sendld[0], g_detailtoke[:-4]+'%3D', sign_key)
        # LOLLLLLLLLLLLLLLLLLLLLLLLLL
        time.sleep(1)
        receive_content = requests.get(receive_url, headers=headers_fake).content

        try:
            return_json_data = json.loads(receive_content)
            if return_json_data['retmsg'] == 'ok':
                money_count = return_json_data['amount'] * 0.01
                print '[+] Success: %s$' % money_count
                return money_count
            else:
                print '[-] Status: %s' % return_json_data['retmsg']
        except Exception, e:
            print str(e)

#### End

文中部分代码和图片因时间无法全面提供，修复后不再传输红包地址：

![http://ww2.sinaimg.cn/large/c334041bjw1f1cqbpavrqj20jk0k2ae3.jpg](http://ww2.sinaimg.cn/large/c334041bjw1f1cqbpavrqj20jk0k2ae3.jpg)

如果你要问现在还有没有新的方法，嗯是有的。

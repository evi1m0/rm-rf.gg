---
layout: post
title: wafCheck.py DEMO - Hook urllib2 / requests
---
#### 0x01 描述

这个 DEMO 是当时使用 [Beehive](https://github.com/n0tr00t/Beehive) 时所编写的辅助脚本（近期整理文件所发现），在攻击或测试脚本运行时 Hook 住所用到的网络请求库，调用 wafCheck.py 来判断是什么原因导致 Payload 失效，当时大致分为：漏洞不存在或被防火墙拦截。

#### 0x02 WafCheckSource

    #!/usr/bin/env python
    # coding=utf8
    # author=evi1m0@2014
    
    import urlparse
    
    '''
    Check WAF:
        1. 360
        2. JIASULE
        3. BAIDUYUN
        4. ANQUANBAO
        5. ...
    '''
    
    baidu_waf_list = [('&#24403;&#21069;&#35775;&#38382;&#30097;&#20284;&#40657;&#23458;&#25915;'
                       '&#20987;&#65292;&#24050;&#34987;&#30334;&#24230;&#20113;&#21152;&#36895;'),
                      '<a id="official_site" href="http://yunjiasu.baidu.com" target="_blank">',]
    
    wzws_waf_list = ['<p style="color:#686868;line-height:30px;margin:0px;">',
                    'location.href="http://wangzhan.360.cn/index/shouquan/host/"+host+"/?url="+url;',
                    '<img src="/wzws-waf-cgi/wzblogo.png" border="0">',]
    
    jsl_waf_list = ["ShtwNRFkQSxxST4/XYS1xy6riMRYsTL75Hq38joEXZrPEVm11HaOwd5HTt",
                    '&#24403;&#21069;&#35775;&#38382;&#30097;&#20284;&#40657;&#23458;&#25915;&#20987;&#65292',]
    
    anquanbao_waf_list = ['http://www.anquanbao.com/operatedomain.x?x=enter_green_passage&greenpassage=',
                          'background: url("/aqb_cc/error/img/new_submit.png") 0 0 no-repeat;',
                          'href="http://www.anquanbao.com/green-passage/" target="_blank" class="greenpassage"',
                          '<div class="err_tips">',]
    
    check = lambda x, a: x in a
    def _waf(url, response):
        target = urlparse.urlparse(url).netloc
        color_start = '\033[1;31;40m'
        color_end = '\033[0m'
    
        # Baidu WAF
        result = [check(i, response) for i in baidu_waf_list]
        if all(result):
            print '%s[+] %s: Baidu WAF%s' % (color_start, target, color_end)
    
        # 360 WAF
        result = [check(i, response) for i in wzws_waf_list]
        if all(result):
            print '%s[+] %s: 360 WAF%s' % (color_start, target, color_end)
    
        # JSL WAF
        result = [check(i, response) for i in jsl_waf_list]
        if all(result):
            print '%s[+] %s: JSL WAF%s' % (color_start, target, color_end)
    
        # ANQUANBAO WAF
        result = [check(i, response) for i in anquanbao_waf_list]
        if all(result):
            print '%s[+] %s: AnQuanBao WAF%s' % (color_start, target, color_end)

#### 0x03 Hook

- E.g MacOSPlat

urllib2 path: /System/Library/Frameworks/Python.framework/Versions/2.*/lib/python2.7/urllib2.py

    123~134Line

    def urlopen(url, data=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT):
        global _opener
        if _opener is None:
            _opener = build_opener()
        # hooked by evi1m0
        import wafCheck
        try:
            _response = _opener.open(url, data, timeout).read()
        except HTTPError, e:
            _response = e.read()
        wafCheck._waf(url, _response)
        return _opener.open(url, data, timeout)
    
    204~233Line
    
    class Request:
    
    def __init__(self, url, data=None, headers={},
                 origin_req_host=None, unverifiable=False):
        # unwrap('<URL:type://host/path>') --> 'type://host/path'
        self.__original = unwrap(url)
        self.__original, self.__fragment = splittag(self.__original)
        self.type = None
        # self.__r_type is what's left after doing the splittype
        self.host = None
        self.port = None
        self._tunnel_host = None
        self.data = data
        self.headers = {}
        for key, value in headers.items():
            self.add_header(key, value)
        self.unredirected_hdrs = {}
        if origin_req_host is None:
            origin_req_host = request_host(self)
        self.origin_req_host = origin_req_host
        self.unverifiable = unverifiable
    
        # hooked by evi1m0
        import urllib
        import wafCheck
        try:
            _response = urllib.urlopen(url, self.data).read()
        except HTTPError, e:
            _response = e.read()
        wafCheck._waf(url, _response)

requests path: /Library/Python/2.7/site-packages/requests-2.*/requests/api.py

    def request(method, url, **kwargs):
        """Constructs and sends a :class:`Request <Request>`.
        Returns :class:`Response <Response>` object.
    
        :param method: method for the new :class:`Request` object.
        :param url: URL for the new :class:`Request` object.
        .....
        :param verify: (optional) if ``True``, the SSL cert will be verified. A CA_BUNDLE path can also be provided.
        :param stream: (optional) if ``False``, the response content will be immediately downloaded.
        :param cert: (optional) if String, path to ssl client cert file (.pem). If Tuple, ('cert', 'key') pair.
    
        Usage::
    
          >>> import requests
          >>> req = requests.request('GET', 'http://httpbin.org/get')
          <Response [200]>
        """
    
        session = sessions.Session()
    
        # hooked by evi1m0
        import wafCheck
        _response = session.request(method=method, url=url, **kwargs).text
        wafCheck._waf(url, _response)
    
        return session.request(method=method, url=url, **kwargs)

#### 0x04 Usage*Demo

上面将 requests 及 urllib2 的请求代码 hook 以后调用编写的 wafCheck.py 脚本，这里将 wafCheck cp 到对应的目录即可，如：

    ➜  Desktop cp wafCheck.py /System/Library/Frameworks/Python.framework/Versions/2.*/lib/python2.*/urllib2.py
    ➜  Desktop cp wafCheck.py /Library/Python/2.*/site-packages/requests-2.*/requests/api.py

现在进行网络请求时倘若被防火墙拦截，上面的脚本便可以清晰的告诉我们是哪个规则将我们拦截住，防止漏掉原本存在的漏洞：

![http://ww2.sinaimg.cn/large/c334041bjw1f3i8i52m3cj20rx0iqacq.jpg](http://ww2.sinaimg.cn/large/c334041bjw1f3i8i52m3cj20rx0iqacq.jpg)
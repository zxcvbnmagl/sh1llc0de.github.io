---
layout:     post
title:      实验吧_题目总结 
subtitle:   实验吧
date:       2018-08-10 				# 时间
author:     sh1llc0de					# 作者
catalog: true 						# 是否归档
tags:								#标签
    - CTF
---

# 实验吧web题wp
>前段时间花了点时间做了一下，学到了蛮多东西。  
[题目](/file/shiyanba.rar)

## web
* ### web_1
	一道代码审计的题目，思路也满清晰的，一路跟下去就可以知道大概怎么做。具体的解法其实从第一个if就已经知道了。通过那个判断我们可以知道可以/web1.php/web1.php这样传参。然后我们就可以拿到Flag了。
* ### web_2
	首先extract函数和trim函数都是没碰到过的。所以我们先学习一下，然后通过菜鸟教程知道了extract函数是一个把数组的键当成变量名，值为变量的值。一般为变量覆盖漏洞。然后trim函数是去除头尾的字符。知道这些我们就可以做题了。对了，还有一个file_get_contents()函数，这个函数是把文件当成字符串。首先我们想到了变量覆盖，直接覆盖掉$file，然后我们就给$shiyan传递一个空值就行，这样的话，Flag就拿到了。
* ### web_3
	serialize函数，这个是php中的序列化操作的函数，就是把对象的状态信息转换成可以存储或传输形式的过程。讲一讲实例吧，毕竟这个东西非常重要。
	![](https://s1.ax1x.com/2018/08/10/P6pwFg.png)
	这是最简单的序列化操作，我们来看一下结果。
	![](https://s1.ax1x.com/2018/08/10/P6pBWj.png)
	我来解释一下，首先第一个a代表的是这是一个数组经过序列化操作得到的，3代表的是这个数组是有3个元素，然后花括号里面的东西就是我们定义的键值对了。s代表的是字符串的意思，1就是它的长度，后面就是它的值了。接下来的东西就是它的键对应的值。
	
	接下来讲一下反序列化操作，还是用举例子的方法来说明。
	![](https://s1.ax1x.com/2018/08/10/P6p0YQ.png)
	通过反序列化函数unserialize把它给反转回来。看看结果。
	![](https://s1.ax1x.com/2018/08/10/P6polR.png)
	这就是序列化和反序列化。然后我们来看一下题目。
	我们仔细观察可以发现这里使用==来判断的，==是一种弱类型的比较。所以我们可以绕过，当它和一个bool类型的数据进行比较时，也会转换成bool类型，而计算机遵循的规则是非0即真，所以我们只有传递一个true就行了。payload的构造的话可以看看我们前面的格式。

	payload:   ?info=a:2:{s:8:"username";b:1;s:8:"password";b:1;}
* ### web_4
	这题是一题送分题了，未必也太简单了。首先判断是否为空，然后是一个正则匹配，匹配的内容是[a-zA-Z0-9]，然后判断一下长度是否小于8并且数字大小大于99999999，最后判断有没有包含*_*这个字符。
	这题的思路也蛮简单的，首先从正则表达式可以知道我们可以传递字母，所以想到了科学计数法。我们就能绕过那个<8>99999999的操作了。接下来是要绕过那个*_*了，这里就很好玩了，因为ereg有一个漏洞，就是当我们使用%00来截断的话，那么他就不会对比后面的值了，所以payload为：9e9%00*_*
* ### web_5
	这题的话比上面的那题还要简单，也不知道那个github的作者是怎么排序的，可能是随便吧(哈哈哈)，我们仔细(瞄一眼就行)观察题目可以发现用的还是==这种弱类型的比较方式，不对劲，这题应该不管它的事，这题应该这样的，就是strcmp函数有一个缺陷，在php5.2之前是会报错的，5.2之后的版本里有直接返回0，虽然5.2会报错，但还是会判断相等。所以我们可以传递一个数组就行绕过。
	payload: ?a[]
* ### web_6
	这题有两种做法，第一种是在第一次访问的前提下进行的，你也可以开隐私窗口访问，原理是这样的。我们在第一次访问的时候，是没有PHPSESSID的，然后我们只要传一个空值的password就行了。如果是第二次访问，PHPSESSID是存在的，但是我们可以通过burp删除，然后也可以绕过。
* ### web_7
	这题也是一题经常出现的题目了，这个还是科学计算法的原因才导致了我们可以绕过。我们只要找一个经过sha1操作之后的字符是0e开头的就行。我这里记录一下，以后反正也会用到。首先sha1的是
	aaroZmOk  |  aaK1STfY。md5的是QNKCDZO  |  s878926199a。接下来是md5生日攻击
	Param1=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7
	%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3
	%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f
	%07%fe%a2
	Param2=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7
	%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3
	%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f
	%07%fe%a2  
	还有一种做法是传两个数组，这个听说也能绕过md5什么的。payload:username[]=1&password[]=2
* ### web_8
	这题的话是给了一个加密算法和加密过后的字符串，要我们自己来实现解密，我想是python和php都实现一下,python的话就当是复习了。先来python的吧。
	经过本人一番挣扎，最后还是写出来了这个脚本，写的过程中碰到了蛮多的问题。。。。我先去写一下注释在贴过来。
	```python
import base64
def python_decode(string):
    zimu = "abcdefghijklmnopqrstuvwxyz"
    rot_13 =""
    for i in string:
        if i.isdigit():
            rot_13 += i
        else:
            try:
                rot_13 += zimu[zimu.index(i)-13]
            except:
                rot_13 += zimu[zimu.index(i.lower())-13].upper()
                #以上操作都是那个啥来着，rot13
    fz = rot_13[::-1]
    #字符串反转
    base = base64.b64decode(fz)
    #base64解密
    base = [chr(ord(i)-1) for i in base]
    #这里是我进步的地方，我直接chr(ord(i)-1)了，原来的我肯定会for循环什么的。
    fz = base[::-1]
    #字符串反转
    print "".join(fz)
    #数组拼接
 
python_decode("a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws")
	```
	注释的是不是很好，嘿嘿。不管了，还是很简单的。接下来学一下php，php其实容易的多，都有内置函数
	```php
<?php
function decode($str){
$a = str_rot13($str);
$b = strrev($a);
$c = base64_decode($b);
for ($d=0;$d<strlen($c);$d++){
$e = substr($c,$d,1);
$f = chr(ord($e)-1);
$g = $g.$f;
}
$h = strrev($g);
echo $h;
}
 
decode("a1zLbgQsCESEIqRLwuQAyMwLyq2L5VwBxqGA3RQAyumZ0tmMvSGM2ZwB4tws");
?>
	```
	哇，还是php容易啊，全是内置函数，我他妈的python写了差不多半个小时。。太菜了。不过还是有进步。
* ### web_9
	这题的环境是要手动改的，比如数据库账号密码什么的。改完之后我们才不多就可以开始做题了。这题非常的基础，就是一个普通注入，通过审计发现，我们提交的user要和md5过的pass相同，本来是很难实现的，但是这里存在sql注入漏洞，我们可以构思sql语句来绕过。payload: user=-1' union select "c4ca4238a0b923820dcc509a6f75849b"  %23&pass=1，这里还有一个小问题，就是如果我们是在输入框里面提交的话，那么%23就要变成#号了。
* ### web_10
	这题也非常的容易，原理是我们提交的值不能为hackerDJ,但是解码之后又要为hackDJ，然后我们就把他编码一次在提交，因为是在浏览器的地址框里面输入的，它也会默认接一次码，我们通过2次编码就可以绕过了。payload:?id=%2568%2561%2563%256b%2565%2572%2544%254a
* ### web_11
	这题和第九题差不多，我这里由于环境的原因所以有点问题，但是这题还是普通注入题。给个payload吧
	payload:user=admin') union select "admin" %23pass=1
* ### web_12
	先审计一下（有源码的前提下），发现了是通过各种方式来获取我们的ip，然后进行判断是否为1.1.1.1,用XFF头可以绕过去。下面不是我的电脑啊，我可买不起，我懒的自己操作一边，copy过来的。
	```
GET /shiyanba/web12.php HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:54.0) Gecko/20100101 Firefox/54.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
X-forwarded-for: 1.1.1.1
Upgrade-Insecure-Requests: 1
 
	```
* ### web_13
	诡异的发现没有web13。。。。。。
* ### web_14
	这题不知道什么原因，一直没做出来，后面直接用作者给的payload都不行，后面就尝试着自己去在往下绕过，最后还是出来了。首先放作者的提示吧。
	```
 
mysql> select * from  user ;
+----+-----------+----------------------------------+-------+
| id | uname     | password                         | level |
+----+-----------+----------------------------------+-------+
|  1 | admin     | 21232f297a57a5a743894aoe4a801fc3 |     1 |
|  2 | test      | 202cb962ac59075b964b07152d234b70 |     0 |
|  3 | wonderkun | e10adc3949ba59abbe56e057f20f883e |     0 |
|  4 | 想放假    | e10adc3949ba59abbe56e057f20f883e |     0  |
+----+-----------+----------------------------------+-------+
 
mysql> select * from  user where 1=1  group by  password   ;
+----+-----------+----------------------------------+-------+
| id | uname     | password                         | level |
+----+-----------+----------------------------------+-------+
|  2 | test      | 202cb962ac59075b964b07152d234b70 |     0 |
|  1 | admin     | 21232f297a57a5a743894aoe4a801fc3 |     1 |
|  3 | wonderkun | e10adc3949ba59abbe56e057f20f883e |     0 |
+----+-----------+----------------------------------+-------+
 
mysql> select * from  user where 1=1  group by  password  with rollup limit 1 offset 3 ;
+----+-----------+----------+-------+
| id | uname     | password | level |
+----+-----------+----------+-------+
|  3 | wonderkun | NULL     |     0 |
+----+-----------+----------+-------+
//上面就是构造空密码用户的方法。
	```
	这就是作者给的，然后我自己有联合了mysql的一些位运算符号才做出来，首先上面的方法在一定情况下是可行的。我是用来异或操作payload: ' ^ 0  group by password with rollup limit 1 offset 3#
* ### web_15
	待更新  

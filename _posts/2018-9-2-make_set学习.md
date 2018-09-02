---
layout:     post
title:      make_set学习
subtitle:   sql
date:       2018-09-02 				# 时间
author:     sh1llc0de					# 作者
catalog: true 						# 是否归档
tags:								#标签
    - SQL
---

MAKE_SET(bits,str1,str2,...)
返回一个设定值 (一个包含被‘,’号分开的字字符串的字符串) ，由在bits 组中具有相应的比特的字符串组成。str1 对应比特 0, str2 对应比特1,以此类推。str1, str2, ...中的 NULL值不会被添加到结果中。

```mysql
mysql> SELECT MAKE_SET(1,'a','b','c');

-> 'a'

mysql> SELECT MAKE_SET(1 | 4,'hello','nice','world');

-> 'hello,world'

mysql> SELECT MAKE_SET(1 | 4,'hello','nice',NULL,'world');

-> 'hello'

mysql> SELECT MAKE_SET(0,'a','b','c');

-> ''
```

bits应将期转为二进制，如，1为，0001,倒过来排序，则为1000,将bits后面的字符串str1,str2等，放置在这个倒过来的二进制排序中，取出值为1对应的字符串，则得到hello

1|4表示进行或运算，为0001 | 0100,得0101，倒过来排序，为1010，则'hello','nice','world'得到的是hello word。'hello','nice',NULL,'world'得到的是hello。null不取，只有1才取对应字符串

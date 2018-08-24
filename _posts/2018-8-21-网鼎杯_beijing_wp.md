---
layout:     post
title:      beijing_wp
subtitle:   beijing
date:       2018-08-21 				# 时间
author:     sh1llc0de					# 作者
catalog: true 						# 是否归档
tags:								#标签
    - CTF
---

>这次打网鼎杯给我的感觉就是全场被虐，web狗无法存活。

## 运行分析
	直接运行发现打印了一些乱码的字符串
	![](https://s1.ax1x.com/2018/08/21/PILX6g.png)

## ida一把梭
	先查看函数发现有main函数，那就直接f5,f5,f5。
	![](https://s1.ax1x.com/2018/08/21/PIOKtx.png)

	之后我们发现是直接调用sub_8048460函数并打印返回值
	![](https://s1.ax1x.com/2018/08/21/PIOl9K.png)
	
	然后我们就查看这个传入的值是什么,发现是一些数字啥的。
	![](https://s1.ax1x.com/2018/08/21/PIOtHA.png)

	接下来是查看这个sub_8048460函数了。
	![](https://s1.ax1x.com/2018/08/21/PIOd4P.png)
	
	发现有一个分支，于是猜测可能是flag的长度。
	![](https://s1.ax1x.com/2018/08/21/PIOB38.png)

	接下来随便跟进一个分支进行分析,可以发现是一个亦或操作，我们在去看看要被亦或的值。都是上面的值亦或下面的值。因为发现差不多都是可打印字符串，
	所以再次猜测flag在这里，但是顺序是乱的。而原本的顺序可能是在main里面的list中。
	![](https://s1.ax1x.com/2018/08/21/PIO6Bj.png)
	![](https://s1.ax1x.com/2018/08/21/PIO43T.png)
	
	我们去main下面的list中提取数据
	![](https://s1.ax1x.com/2018/08/21/PIxV8x.png)

	之后尝试写一下脚本。先理一下思路。
	```c
	# 伪代码
	encode():
		return flag[i] ^ xor[i]
	main():
		list[] < = 记录着正确的 flag 打印顺序 
		print encode(list[i])
		
		
	# python
	string = 'aginbefjml{z}_'
	num = [6, 9, 0, 1, 0xa, 0, 8, 0, 0xb, 2, 3, 1, 0xd, 4, 5, 2, 7, 2, 3, 1, 0xc]
	flag = ''
	for i in num:
		flag += string[i]
	print(flag)
	```
## flag
	![](https://s1.ax1x.com/2018/08/21/PIztYR.png)

---
ahtuor: Fish
layout: post 
title: 一堆数的最小公倍数--python
categories: Python
tags: algorithm, acm
---
#问题描述：
> 给你一个正整数list L, 如 L=[2,8,3,50], 求列表中所有数的最小公倍数(不用考虑溢出问题）
><br/> 如L=[3,5,10], 则输出30

#思路分析：
参考博文[ 趣题：不用除法，如何求n个数的最小公倍数 ](http://www.cnblogs.com/Cwdf/archive/2010/05/13/1734786.html), 思路就是每行的最小数字都加上自己，直到都相等为止。如下例子：
<!--more-->
{% highlight c %}
30 12 18
30 24 18
30 24 36
30 36 36
60 36 36
60 48 36
60 48 54
60 60 54
60 60 72
90 60 72
90 72 72
90 84 72
90 84 90
90 96 90
120 96 90
120 96 108
120 108 108
120 120 108
120 120 126
150 120 126
150 132 126
150 132 144
150 144 144
150 156 144
150 156 162
180 156 162
180 168 162
180 168 180
180 180 180
{% endhighlight %}
##*代码如下*
{% highlight  python %}
L = [2, 5, 6, 7]
b = [i for i in L]
length = len(L)
def all_equal(ilist):
	start  = ilist[0]
	for key in ilist[1:]:
		if start != key:
			return False
	return True
while not all_equal(L):
	min_num = min(L)
	for index, item in enumerate(L):
		if item == min_num:
			L[index] += b[index]
print L[0]
{% endhighlight %}

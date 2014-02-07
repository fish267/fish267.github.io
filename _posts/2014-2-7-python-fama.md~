---
author: Fish
layout: post 
categories: Python
tags: python, algorithm
title: 砝码称重问题-python
---
<br>
#问题描述


>有一组砝码，重量互不相等，分别为m1、m2、m3……mn；它们可取的最大数量分别为x1、x2、x3……xn。 
>现要用这些砝码去称物体的重量,问能称出多少种不同的重量。 
>现在给你两个正整数列表w和n， 列表w中的第i个元素w[i]表示第i个砝码的重量，列表n的第
>i个元素n[i]表示砝码i的最大数量。i从0开始，请你输出不同重量的种数。
>如：w=[1,2], n=[2,1], 则输出5（分析：共有五种重量：0,1,2,3,4）
-----
#解体思路
<!--more-->
使用Python的set，可以保证唯一性，还有重量为0的情况
i
{% highlight python %}
cnt = set()
w = [2, 3, 5, 6, 8, 9, 11]
n = [5, 2, 6, 7, 1, 3, 14]
for i in range(n[0] + 1):
	cnt.add(w[0] * i)
for k in range(1, len(w)):
	temp = cnt.copy()
	cnt.clear()
	for j in range(n[i] + 1):
		for k in temp:
			cnt.add(w[i] * j + k)
print len(cnt)
{% endhighlight %}	

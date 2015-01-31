---
author: Fish
layout: post
title: 快速素数产生
categories: Python
tags: algorithm acm
---
在ACM会经常遇到使用大量素数的情景，谷歌了一下，当不是特别多时，可以使用**筛选法**搞定.
这个[wiki](http://zh.wikipedia.org/wiki/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95)上扒来的原理图:<br>
![pic](http://upload.wikimedia.org/wikipedia/commons/b/b9/Sieve_of_Eratosthenes_animation.gif)
<br>
   埃拉托斯特尼筛法，简称埃氏筛，是一种公元前250年由古希腊数学家埃拉托斯特尼所提出的一种简单检定素数的算法。
<!--more-->
<br>
      下面是伪代码:
{% highlight c %}
// arbitrary search limit
limit ← 1.000.000             
 
// assume all numbers are prime at first                                   
 
is_prime(i) ← true, i ∈ [2, limit]
 
for n in [2, √limit]:
    if is_prime(n):
        // eliminate multiples of each prime,
        // starting with its square
        is_prime(i) ← false, i ∈ {n², n²+n, n²+2n, ..., limit}
 
for n in [2, limit]:
    if is_prime(n): print n
{% endhighlight %}
***针对上面的思路，Python代码如下***
{% highlight python %}
prime_dic = {}
prime_list = []
n = 1000
for i in range(2, n + 1):
	prime_dic[i] = 1
for i in range(2, int(n ** .5) + 1):
	for j in range(i ** 2, n + 1, i):
		if prime_dic[i] == 1:
			prime_dic[j] = 0
for k, v in prime_dic.items():
	if v == 1:
		prime_list.append(k)
print prime_list
{% endhighlight %}

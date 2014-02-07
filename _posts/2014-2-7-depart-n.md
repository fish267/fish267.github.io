---
author: Fish
layout: post
title: 因式分解n!--python
categories: Python
tags: algorithm
---
<br>
# 问题描述
	给你一个数 n (1 < n <= 1000000) ,求 n! (n的阶乘)的质因数分解形式.
	质因数分解形式为:
	n=p1^m1*p2^m2*p3^m3……
	* 这里 p1 < p2 < p3 < …… 为质数
	* 如果 mi = 1, 则   ^ mi   就不需要输出 
	如：n=6,则输出：6=2^4*3^2*5
        n=7,则输出：7=2^4*3^2*5*7
<!--more-->
##*代码如下：*
{% highlight python %}
prime_dic = {}
prime_list = []
n = 10
for i in range(2, n + 1):
    prime_dic[i] = 1
for i in range(2, int(n ** .5) + 1):
    for j in range(i * i, n + 1, i):
        if prime_dic[i] == 1:
            prime_dic[j] = 0
for k,v in prime_dic.items():
    if v == 1:
        prime_list.append(k)
#print prime_list
 
def count_prime(n, k):
    num = 0
    while(n):
        num += n/k
        n /= k
    return num
result = '%d=' %n
for i in range(2, n + 1):
    if i in prime_list:
        times = count_prime(n, i)
        if times > 1:
            result += '%d^%d*' %(i, times)
        else:
            result += '%d*' %i
print result[:-1]
{% endhighlight %}

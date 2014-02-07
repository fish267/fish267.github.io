---
author: Fish
title: 十进制转化成其他进制
categories: Python
tags: algorithm, acm
layout: post
---
是按正常的手算流程，取余数得来，使用了一个递归,**代码如下：**
{% highlight python %}
dic_alpha = {10: 'A', 11: 'B', 12: 'C', 13: 'D', 14: 'E', 15: 'F' }
converted_list = []
def convert(num, base):
	num = abs(num)
	remainder = num % base
	if num >= base:
		convert(num / base, base)
	converted_list.append(remainder)
def format_print(num, base):
	if num < 0:
		result = '-'
	else:
		result = ''
	convert(num, base)
	
	for item in converted_list:
		if item in dic_alpha.keys():
			item = dic(item)
			
		result += str(item)
	print result
{% endhighlight %}

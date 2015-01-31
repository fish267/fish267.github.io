---
author: Fish
layout: post
title: Windows下hosts更新 
categories: windows 
tags: python
---

得益于这个网站的更新，http://www.360kb.com/kb/2_122.html


手写了一个脚本，就不必手动复制粘贴了，需要设置一下hosts的权限，否则无法修改。


代码：
 
{% highlight python %}
import sys, os
import urllib.request, re

url = "http://www.360kb.com/kb/2_122.html"

html_content = urllib.request.urlopen(url).read()

frase = r'#base services.*2015 end'
ip = re.search(frase, str(html_content)).group()
ip = ip.replace("&nbsp;", ' ')
ip = ip.replace("<br />", '')

f = open('C:\Windows\System32\drivers\etc\hosts', 'w')
f.write('')
f.close()
for t in ip.split('\\n')[:-10]:
    f = open('C:\Windows\System32\drivers\etc\hosts', 'a+')
    f.write(t + '\n')
    f.close()


f = open('C:\Windows\System32\drivers\etc\hosts', 'r')
    print(f.read())

{% endhighlight %}

#END

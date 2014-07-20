---
author: Fish
layout: post
title: Python 处理字符编码 
categories: Python
tags: charset 
---

#参考文章
主要参考了这两篇文章，写的非常好:

[unicode in python](http://farmdev.com/talks/unicode/)


[详解 python 中文编码与处理](http://my.oschina.net/leejun2005/blog/74430)

##总结如下
处理文本或者抓取的网页时，尽量这样的初始化：

{% highlight python %}
def unicode_init(obj, encoding = 'utf-8'):
    if isinstance(obj, basestring):
        if not isinstance(obj, unicode):
            obj = unicode(obj, encodeing)
    return obj

{% endhighlight %}

这么处理，是先将不同的编码格式，统一成unicode,然后在输出的时候，再变回去。

<!--more-->
##举个例子
比如下面这个页面，如果直接读取并输出到命令窗，会有堆乱码
{% highlight python %}
import urllib2
url = 'http://tech.sina.com.cn/t/2014-07-20/10009505866.shtml'
a = urllib2.urlopen(url).read()
{% endhighlight %}
输出结果：

    *2\x97\x13)S\x1f\x11F\x05\x8f-HW\xf3 J\x9eH\xe8\x1fO

因为那个界面的编码是'gb2312'格式的，所以输出会出乎意料.


这样使用:
{% highlight python %}
url = 'http://tech.sina.com.cn/t/2014-07-20/10009505866.shtml'
a = urllib2.urlopen(url).read()
print unicode_init(a, encoding='gb2312') 
{% endhighlight %}

#END


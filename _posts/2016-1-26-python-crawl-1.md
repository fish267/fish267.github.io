---
author: Fish
layout: post
title: Python 简单爬虫
categories: python
tags: linux
---

## 1. 什么是爬虫

 [爬虫](https://en.wikipedia.org/wiki/Web_crawler)指的系统性抓取网络页面的网络机器人。原理图如下:

 ![crawler](https://upload.wikimedia.org/wikipedia/commons/d/df/WebCrawlerArchitecture.svg)

 上图涉及了爬虫的任务调度控制，多线程，存储机制等，本文不会涉及。

 我们从最简单的故事谈起，当你浏览一个网站时, 发现上面的图片比较有意思，会右键保存下来，当人为操作过多时, 就会想办法自动化实现。 首先是图片的保存， 我们知道直接使用linux 命令 <code>curl -O http://img-url</code>  就可以下载图片;  那么如何获取图片地址呢, 很明显图片的链接地址存放在页面的源码中， <code> command + u </code> 就可以查看到页面的 source code, <code> curl http://site-url </code> 能获取到页面源码， 图片地址一般的格式为 <code> <img src="http://xx.jpg"></img> </code>,
 很容易就能筛选出来。

 上面分为三个部分， 获取页面代码， 解析页面代码， 存储&&展示。因此， 本人对爬虫的理解如下:

 - 打开网页
 - 解析网页
 - 重复上两步，将数据保存下来

回到最开始的爬虫原理图， 在大规模网页爬取时，考虑到性能问题，会引入多线程等集群化抓取, 比如[rq](https://github.com/nvie/rq) ； 已经爬过的页面不能从复爬，考虑到去重复问题， 会引入个hash或者更高级的[Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter); 数据的保存和抽取，丢到 MogoDB 中。 这样上图中的内容都包括了。


## 2. 举个例子

不忘初衷，以爬[妹子图](http://www.meizitu.com) 为例，简单说明一下简单爬虫的需要用到的知识。

{% highlight python %}
import os
from urllib.request import urlopen, urlretrieve
import re
import time
import sys


def get_girls(url):
    page_source = urlopen(url).read().decode('GBK')
    pattern = r'<img alt="\S+" src="(\S+)" /><br />'
    img_urls = re.findall(pattern, page_source)
    for img_url in img_urls:
        img_name = sys.path[0] + os.sep + str(time.time()) + '.jpg'
        urlretrieve(img_url, filename=img_name)

get_girls('http://www.meizitu.com/a/5284.html')
{% endhighlight %}

函数 <code>get_girls</code> 里面，使用到了 urllib 这个库与 re 正则库, urlopen() 将页面源码拉下来， re.findall(pattern, source) 匹配到图片地址， urlretrieve() 下载到本地。 很简单吧~ 如果只是获取到图片地址，可直接在交互命令行搞

    >>> url = 'http://www.meizitu.com/a/5284.html'
    >>> page_source = str(urlopen(url).read())
    >>> img_urls = re.findall(r'<img alt="\S+" src="(\S+)" /><br />', page_source)
    >>> img_urls
    ['http://pic.meizitu.com/wp-content/uploads/2016a/01/21/01.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/02.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/03.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/04.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/05.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/06.jpg', 'http://pic.meizitu.com/wp-content/uploads/2016a/01/21/07.jpg']

页面源码拉取， 解析html, 获取数据，以上就是简单 Python 爬虫介绍。针对解析这一部分，有很多开源的库可以玩，非常有意思~

## 2.1  Python urllib 库

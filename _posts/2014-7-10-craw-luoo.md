---
author: fish
layout: post
title: 下载落网歌曲和图片
categories:
- Python
tags: Linux
---
**说了话长，长话短说**，在上一篇文章的脚本跑到落网第65期时，网站就更新了，以前能同过解析网页源码获取到 mp3 和图片的下载地址，现在不行了， 落网给他加密了！！！


有点不爽，于是趁着今天在公司加班的间隙，试着搞定它，奈何实在无法忍受阅读落网压缩过后的 js 代码。 终于憋尿的情景下， 想到了落网以前正常的 mp3 下载地址格式。 

    http://luoo.800edu.net/low/luoo/radio1/2.mp3
    
这下就有的搞了，分析了一下， 前几十期的歌曲名是 1.mp3 2.mp3 ，后面的都规范为 01.mp3, 02.mp3 了， 图片下载吗， 还是跑到网站上抓 url 了， 总不能连图片地址对浏览器也是加密吧！！


回去了， 今天华为的一个朋友上海回来，一起吃饭吐槽生活， 抱怨社会吧， 哈哈哈~~


<!--more-->
贴上代码, 这个是下载歌曲的， 从下载图片的过程中学习了一下 zip 函数的使用， python 真是太好玩了~ 
{% highlight python %}
#coding:utf-8
import urllib2
import re
import os
import json
import threading
import logging
log_file = 'f:\music\log\music.log'
# log_file = '/home/fish/log/download.log'
logger = logging.getLogger()
logger.addHandler(logging.FileHandler(log_file))
logger.setLevel(logging.DEBUG)


mp3URL = 'http://luoo.800edu.net/low/luoo/radio601/02.mp3'
# create 01 02..
# 对于前几期，是1.mp3, 2.mp3...
def createRange(m, n):
    urlList = []
    for i in range(m, n):
        i = str(i)
        if len(i) == 1:
            i = '0' + i
        urlList.append(i)
    return  urlList
#  
def getURL(vol):
    url = 'http://luoo.800edu.net/low/luoo/radio'
    try:
        url += vol 
        jsonList = []
        for item in createRange(1, 20):
            jsonList.append(url + '/' + item + '.mp3')
        return jsonList
    except Exception, e:
        logger.error(e)

def startWork(vol):
    try:
        pwd = os.getcwd()
        if not os.path.exists(vol):
            os.mkdir(vol)
        else:
            volDir = pwd + os.sep + vol
            os.chdir(volDir)
            
            for song in getURL(vol):
                print 'downloading..............' + song
                os.system('wget ' + song)
        os.chdir(pwd)
    except Exception, e:
        logger.error(e)
if __name__ == '__main__':
    for i in range(65, 625):
        startWork(str(i))
{% endhighlight %}

下载图片的：
{% highlight python %}
#coding:utf-8
import urllib2
import re
import os
from bs4 import BeautifulSoup as bs
import logging
log_file = 'f:\music\log\music.log'
# log_file = '/home/fish/log/download.log'
logger = logging.getLogger()
logger.addHandler(logging.FileHandler(log_file))
logger.setLevel(logging.DEBUG)

# create 01 02..
def getURL(vol):
    url = 'http://www.luoo.net/music/'
    try:
        imgList = []
        url += vol 
        soup = bs(urllib2.urlopen(url).read())
        for div in soup.find_all('div', 'player-wrapper'):
            for img in div.find_all('img'):
                imgList.append(img['src'])
        return imgList
    except Exception, e:
        logger.error(e)
# print getURL('100')
def startWork(vol):
    try:
        pwd = os.getcwd()
        if not os.path.exists(vol):
            os.mkdir(vol)
        else:
            volDir = pwd + os.sep + vol
            os.chdir(volDir)
            # zip 函数的使用，图片保存并重命名
            for song, index  in zip(getURL(vol), range(1, len(getURL(vol)))):
                index = str(index)
                if len(index) == 1:
                    index = '0' + index
                cmd = 'wget -c ' + song + ' -O ' + index + '.jpg'
                os.system(cmd)
        os.chdir(pwd)
    except Exception, e:
        logger.error(e)
if __name__ == '__main__':
    for i in range(65, 625):
        startWork(str(i))
{% endhighlight %}
    

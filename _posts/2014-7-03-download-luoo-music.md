---
author: fish
layout: post
title:  下载落网音乐 
categories: linux 
tags: python 
---
本来是在百度网盘上看到落网的一个音乐下载的，目录看着不爽，也懒得调整了，写了个下载脚本，纯单线程下载


主要得益于落网网站源码中包含了下载地址, 用正则匹配出来就可以了


到是练习了一下logging模块，以后写Python脚本的时候，就得用这类的方式查看了，老是用print 太业余了点



<!--more-->
{% highlight python %}
import urllib2
import re
import os
import json
import threading
import logging

log_file = '/home/fish/python_test/fm/log/download.log'
logger = logging.getLogger()
logger.addHandler(logging.FileHandler(log_file))
logger.setLevel(logging.DEBUG)

# create 001 002..
def createURL():
    urlList = []
    for i in range(1, 626):
        i = str(i)
        if len(i) == 1:
            i = '00' + i
        elif len(i) == 2:
            i = '0' + i
        urlList.append(i)
    return  urlList
#  
def getURL(vol):
    url = 'http://www.luoo.net/music/'
    try:
        url += vol 
        print url
        page = urllib2.urlopen(url).read()
        playPattern = "volPlaylist\s*=\s*(\[\s*\{[\s\S]+?\}\s*\]);"
        playList = re.compile(playPattern).findall(page)[0]
        jsonList= json.loads(playList)
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
                picName = re.compile('(\d+).mp3').findall(song['mp3'])[0]
                songCmd = 'wget ' + song['mp3']
                picCmd = 'wget -c ' + song['poster'] + ' -O ' + picName + '.jpg'
                os.popen(songCmd)
                os.popen(picCmd)
        os.chdir(pwd)
    except Exception, e:
        logger.error(e)
if __name__ == '__main__':
    urlList = createURL()
    for key in urlList:
        startWork(key)
{% endhighlight %}

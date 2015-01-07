---
author: Fish
layout: post
title: python实现Postman 
categories: linux 
tags: python 
---
之前在绿盟科技实习时，就经常在Django后端和Ajax进行大量的通信。后来很多场景需要进行测试，比如Chorme上的Postman插件和Firefox的Restful拓展。

在开源社区上看到了有人写了一个UI界面的，http://www.oschina.net/code/snippet_1438043_44720。但是没有后端代码，参考github上的代码和原来项目中经常使用的手段，修改完成后端，重复发明轮子，以后应该用的到。

<!--more-->

{% highlight python %}

#!/usr/bin/python
# -*- coding: utf-8 -*-
# jhhttp, a simple API tool, support RESTful APIs
import simplejson
import urllib
import urllib.request
import traceback

class RequestWithMethod(urllib.request.Request):
    def __init__(self, url, method, data=None, headers={},\
        origin_req_host=None, unverifiable=False):
        self._method = method
        urllib.request.Request.__init__(self, url, data, headers, origin_req_host,
                unverifiable)

    def get_method(self):
        if self._method:
            return self._method
        else:
            return urllib.request.Request.get_method(self)

class JhhttpError(Exception):
    def __init__(self, value):
        self.value = value
    def __str__(self):
        return repr(self.value)

def _restful_headers(headers=None):
    if headers is None:
        headers = {}
        headers['Content-Type'] = 'application/json'
    else:
        headers['Content-Type'] = 'application/json'
    return headers

def _restful_data(data):
    if type(data) is not str:
        data = simplejson.dumps(data)
    return data

def _prepare_data(data):
    if type(data) is str or data is None:
        return data
    try:
        data = urllib.urlencode(data)
    except(Exception) as e:
        traceback.print_exc()
        raise JhhttpError(e)
    return data

def _open(url, method, data=None, headers=None):
    if headers is None:
        headers = {}
    try:
        req = RequestWithMethod(url, method, data, headers)
        response = urllib.request.urlopen(req)
    except(Exception) as e:
        traceback.print_exc()
        raise JhhttpError(e)
    return response

def _return(response, response_type):
    try:
        content = response.read()
    except(Exception) as e:
        traceback.print_exc()
        raise JhhttpError(e)
    if not content:
        return None
    if response_type is 'json':
        try:
            ret = simplejson.loads(content)
        except(Exception) as e:
            traceback.print_exc()
            raise JhhttpError(e)
    else:
        ret = content
    return ret

def _check(url, data, headers):
    if headers is not None and type(headers) is not dict:
        raise JhhttpError('headers is not a dict')

def _do_http(url, data=None, headers=None, method='GET', response_type='text'):
    try:
        _check(url, data, headers)
        f = _open(url, method, data, headers)
        ret = _return(f, response_type)
    except(JhhttpError)as e:
        print('JhhttpError, url = %s' % url)
        ret = None
    return ret

def send(method, url, data = None, headers = None):
    method = method.upper()
    if method == 'GET':
        data = None
    else:
        data = _prepare_data(data)
    return _do_http(url, data, headers, method, 'text')
def rest_send(method, url, data = None, headers = None):
    method = method.upper()
    headers = _restful_headers(headers)
    if method == 'GET':
        data = None
    else:
        data = _restful_data(data)
    return _do_http(url, data, headers, method, 'json')
#-------------test------------#
print(rest_send('get', "http://ip.taobao.com/service/getIpInfo.php?ip=124.31.34.190"))

{% endhighlight %}


#END

---
author: Fish
layout: post
title:  Bottle 源码阅读(六) -- Python Web 服务基础
categories: python
tags: system_design
---

![](https://gw.alipayobjects.com/zos/rmsportal/ImavrAPmAViBpPIjIxAP.png)

# 1. Python socket 模块使用

[Python Socket 编程详细介绍](https://gist.github.com/kevinkindom/108ffd675cb9253f8f71) 这篇文章讲解的非常好.

先看下面一段代码: 

<!--more-->

```python 
import socket

HOST, PORT = '', 8888

listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
listen_socket.bind((HOST, PORT))
listen_socket.listen(1)
print('Serving HTTP on port %s ...' % PORT)
while True:
    client_connection, client_address = listen_socket.accept()
    request = client_connection.recv(1024)
    print(type(request))
    print(request)

    http_response = b"""\
HTTP/1.1 200 OK

Hello, World!
"""
    client_connection.sendall(http_response)
    client_connection.close()
``` 


浏览器访问 `localhost:8888` 会展示出 'Hello, World!'.

一般 TCP 服务器设计的步骤如下:

- 创建 socket, 绑定 ip 和 port  `socket() bind()`
- 设置监听链接  `listen()`
- 循环, 监听客户端请求 `while True: listen_socket.accept()`
- 接收请求 `recv()`
- 返回请求 `sendall()`
- 关闭连接 `close()`

客户端链接步骤:

- 创建 socket
- connect()
- sendall()
- recv()
- close()

上面例子代码中, 浏览器访问地址, 就是模拟了客户端的操作, 注意到 http_response 返回内容:

```shell
HTTP/1.1 200 OK

Hello, World!
```

内容包括3个部分: 

1. HTTP version
2. HTTP status code
3. HTTP response body

# 2. 使用 socket 实现一个 WebServer

在重复造轮子前, 先回顾一下 server 怎么使用的:

```python
server = XXServer(ip, port, callbac k)
server.start()
```

也就是说, XXServer 需要将上面的代码包装一层. 实现代码如下: 

```python 
class SixServer(object):
    def __init__(self, host='127.0.0.1', port=8888, callback=None):
        self.host = host
        self.port = port
        self.callback = callback

    def _init_socket(self):
        import socket
        self.listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.listen_socket.bind(('', self.port))
        self.listen_socket.listen(5)

    def start(self):
        self._init_socket()
        print('Serving HTTP on port %s:%s ...' % (self.host, self.port))
        while True:
            self.client_connection, client_address = self.listen_socket.accept()
            try:
                self._handle_request()
                self._handle_response()
            except Exception:
                pass
            finally:
                self.client_connection.close()

    def _handle_request(self):
        raw_request = self.client_connection.recv(1024)
        self._parse_request(raw_request)
        print(self.request)

    def _parse_request(self, raw_request):
        self.request = {}
        # 默认类型是 byte, 需要转码
        raw_request = raw_request.decode('utf-8')
        request_line = raw_request.splitlines()[0]
        # Break down the request line into components
        (self.request['request_method'],  # GET
         self.request['path'],  # /hello
         self.request['request_version']  # HTTP/1.1
         ) = request_line.split()

    def _handle_response(self):
        status = self.request['request_version'] + ' 200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.raw_response = self.callback(self.request)
        # sep 需要输入两个 \n, 一个是不生效的, 怀疑是 encode 的原因.
        sep = '\r\n\n'
        http_response = ''
        # 添加 http 协议 http 状态码
        http_response += status + sep
        # 丰富一下 headers
        for header in response_headers:
            http_response += '{0}:{1}'.format(*header) + sep
        # 添加 回调函数处理后的内容
        for res in self.raw_response:
            http_response = http_response + '{0}: {1}'.format(res, self.raw_response[res]) + sep
       
        self.response = http_response
        print(self.response)
        # http_response = """HTTP/1.1 200 OK\r\n
        # <b>nmhai</b>"""
        self.client_connection.sendall(http_response.encode())
```

## 2.1 代码说明

函数 `_parse_request` 用来处理入参, 比如访问 `http://localhost:8888/cdasfd?key=whoami`, `self.request` 解析如下:

```python 
{'path': '/cdasfd?key=whoami', 'request_version': 'HTTP/1.1', 'request_method': 'GET'}
```

函数 `_handle_response` 用来包装 `response`, 一个标准的 response 上面已经提到过, HTTP 版本号 + HTTP 状态码, 后面跟的分隔符, 要使用 '\r\n\n', 只使用 '\r\n' 浏览器识别不出来.

```python 
# sep 需要输入两个 \n, 一个是不生效的, 怀疑是 encode 的原因.
sep = '\r\n\n'
```

## 2.2 代码调用

首先写个回调函数, 这个函数啥也不做, 直接返回

```python
def callback(text):
    return text
```

调用结果截图如下:

```python
def callback(text):
    return text

six_server = SixServer(callback=callback)
six_server.start()
```

![](https://gw.alipayobjects.com/zos/rmsportal/QNHmCFCmpUGHurPiwkMq.png)


这样, 一个简单的 WebServer 就写完了, 实际使用中, `callback()` 函数主要是拼装 `html` 代码, 类似上面注释掉的两行, 浏览器可以直接将标签渲染出来.

```python 
    http_response = """HTTP/1.1 200 OK\r\n
    <b>nmhai</b>"""
```

# END
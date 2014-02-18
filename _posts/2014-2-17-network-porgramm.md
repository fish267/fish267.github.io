---
author: Fish
layout: post
title: 网络编程笔记
categories: Python
tags: socket, asynchat
---
#socket 模块
在网络编程中的一个基本组件就是**套接字**(socket).套接字主要是两个程序之间的"信息通道"


套接字包括:服务器套接字和客户机套接字, 创建一个服务器套接字后, 就让他等待连接, 这样他就在某个网络地址处(IP地址和一个端口号的组合)监听.<br>


一个套接字就是一个socket模块中的socket类的实例. 他的实例化需要3个参数:
>地址族(默认是<code>socket.AF_INET</code>)<br>
>流套接字(默认是<code>socket.SOCK_STREAM</code>)或数据报套接字<br>
>使用的协议(默认是0)<br>


服务器套接字使用<code>bind</code>方法后, 再调用<code>listen</code>方法去监听这个给定的地址.客户端使用<code>connect</code>方法连接到服务器,在<code>connect</code>方法中使用的地址与<code>bind</code>方法中使用的地址相同. 在这种情况下, 一个地址就是一个格式为<code>(host, port) </code>的元祖.


服务器套接字开始监听后, 就可以接受客户端的连接, 这个步骤使用<code>accept</code>方法来完成. 这个方法会阻塞知道客户端连接, 然后返回一个格式为<code>(client, address)</code>的元祖.


套接字有两个方法: <code>send</code>和<code>recv</code>, 用于数据传输.


###一个小型服务器
{% highlight python %}
import socket

s = socket.socket()
host = socket.gethostname()
port = 5267
s.bind((host, port))
s.listen(5)

while True:
	c, addr = s.accept()
	print "got connect from", addr
	c.send("hello, client!")
	c.close()
{% endhighlight %}
###一个小型客户机
{% highlight python %}
import socket

s = socket.socket()
host = s.gethostname()
port = 5267

s.connect((host, port))
print s.recv(1024)
{% endhighlight %}


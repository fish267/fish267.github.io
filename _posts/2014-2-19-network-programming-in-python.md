---
author: Fish
layout: post
title: Python网络编程(译)
categories: Python
tags: socket tcp
---

这是一个快速学习 Python socket 的教程指南. Python 的socket 编程类似 C 语言.


基本上, sockets 是电脑间网络通信的最基本的元素. 例如, 当你在地址栏中输入 www.love67.net 时, 浏览器会打开并连接一个 socket, 读取 google.com 页面的内容并展示. 同一些聊天客户端例如 Skype, Gtalk 类似, 任何网络通讯都是基于 socket 完成的.


在这个教程中, 我们使用 Python 中的 tcp socket 进行编程, 你也可以参考 [program udp sockets in python](http://www.binarytides.com/programming-udp-sockets-in-python)


#准备知识


本教程假设你已经懂点 Python 的基本知识<br>
让我们开始 socket 的学习吧


#创建一个 socket

我们先创建一个 socket, 使用 <code>socket.socket</code> 来创建:
{% highlight python %}
#Socket client example in python

import socket # for sockets
#create an AF_INET, STREAM_socket(TCP)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Scoket created!'
{% endhighlight %}

函数<code>socket.socket</code>创建一个 socket 并且返回 socket 描述符,  用于其他相关的 socket 函数



上面程序创建 socket 使用了两个参数:

* 地址族: AF_INET(IPV4)
* SOCK_STREAM(是用的是TCP协议)

##错误处理

如果 socket 函数失败了, python 将抛出一个名为 socket.error 的异常, 这个异常必须予以处理:
{% highlight python %}
#handling errors in python socket programs

import socket # for sockets
import sys # for exit

try:
    #create an AF_INET, STREAM_socket(TCP)
    s = scoket.create(socket.AF_INET, socket.SOCK_STREAM)
except socket.error, msg:
    print 'Failed to create socket, Error code: ' + str(msg[0]) 
    print 'Error mesasge: ' + msg[1]
    sys.exit(1)
print 'Socket created!'
{% endhighlight %}
OK, 你已经成功创建了一个 socket, 接下来, 我们将使用这个 socket 连接到 love67.net


##注意


与 SOCK_STREAM 对应的其他类型是 SOCK_DGRAM , 后者是 UDP 协议, 本文我们只讨论 TCP.




#连接到服务器


连接到远程服务器, 需要知道服务器的 IP 地址和端口号, 以 love67.net 的80端口为例.

##首先获取远程服务器的 IP 地址:
{% highlight python %}
import socket 
import sys

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error, msg:
    print 'Failed to create socket, Error code: ' + str(msg[0]) 
    print 'Error mesasge: ' + msg[1]
    sys.exit(1)
print 'Socket created!'

host = 'www.love67.net'
try:
    remote_ip = socket.gethostbyname(host)
except socket.gaierror:
    #coulde not resolve
    print 'Hostname could not be resolved. Exiting...'
    sys.exit()

print 'Ip address of ' + host + ' is ' + remote_ip
{% endhighlight %}

我们获得了 ip 地址后, 接下来连接到指定的端口上:
{% highlight python %}
import socket 
import sys

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error, msg:
    print 'Failed to create socket, Error code: ' + str(msg[0]) 
    print 'Error mesasge: ' + msg[1]
    sys.exit(1)
print 'Socket created!'

host = 'www.love67.net'
port = 80
try:
    remote_ip = socket.gethostbyname(host)
except socket.gaierror:
    #coulde not resolve
    print 'Hostname could not be resolved. Exiting...'
    sys.exit()

print 'Ip address of ' + host + ' is ' + remote_ip

#连接到远程服务器
s.connect((remote_ip, port))
print 'Socket connected to ' + host + ' on ip ' + remote_ip
{% endhighlight %}
运行结果:<br>

    Socket created!
    Ip address of www.love67.net is 199.27.79.133
    Socket connected to www.love67.net on ip 199.27.79.133


创建了一个了一个 socket 并进行连接, 试试其他端口, 你会发现连不上, 这个逻辑可以用来当做端口扫描.



Ok, 连上之后,  我们向服务器发送数据.


##发送数据

函数 <code>sendall</code>用于简单的数据发送, 发送数据到 www.love67.net
{% highlight python %}
import socket	#for sockets
import sys	#for exit

try:
	#create an AF_INET, STREAM socket (TCP)
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error, msg:
	print 'Failed to create socket. Error code: ' + str(msg[0]) + ' , Error message : ' + msg[1]
	sys.exit();

print 'Socket Created'

host = 'www.love67.net'
port = 80

try:
	remote_ip = socket.gethostbyname( host )

except socket.gaierror:
	#could not resolve
	print 'Hostname could not be resolved. Exiting'
	sys.exit()
	
print 'Ip address of ' + host + ' is ' + remote_ip

#Connect to remote server
s.connect((remote_ip , port))

print 'Socket Connected to ' + host + ' on ip ' + remote_ip

#Send some data to remote server
message = "GET / HTTP/1.1\r\n\r\n"

try :
	#Set the whole string
	s.sendall(message)
except socket.error:
	#Send failed
	print 'Send failed'
	sys.exit()

print 'Message send successfully'
{% endhighlight %}
上面代码, 首先连接到目标服务器, 然后发送字符串数据 "GET/HTTP/1.1\r\n\r\n", 这个HTTP协议用来获取网页的内容.

##接收数据
<code>recv</code>用于接收数据
{% highlight python %}
#Socket client example in python

import socket   #for sockets
import sys  #for exit

#create an INET, STREAMing socket
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    except socket.error:
        print 'Failed to create socket'
            sys.exit()
                
                print 'Socket Created'

                host = 'love67.net';
                port = 80;

                try:
                    remote_ip = socket.gethostbyname( host )

                    except socket.gaierror:
                        #could not resolve
                            print 'Hostname could not be resolved. Exiting'
                                sys.exit()

#Connect to remote server
s.connect((remote_ip , port))

print 'Socket Connected to ' + host + ' on ip ' + remote_ip

#Send some data to remote server
message = "GET / HTTP/1.1\r\nHost: love67.net\r\n\r\n"

try :
    #Set the whole string
        s.sendall(message)
        except socket.error:
            #Send failed
                print 'Send failed'
                    sys.exit()

                    print 'Message send successfully'

#Now receive data
reply = s.recv(1000)

print reply
{% endhighlight %}
    Socket Created
    Socket Connected to love67.net on ip 207.97.227.245
    Message send successfully
    HTTP/1.1 200 OK
    Server: GitHub.com
    Date: Tue, 18 Feb 2014 14:08:22 GMT
    Content-Type: text/html; charset=utf-8
    Content-Length: 16284
    Last-Modified: Tue, 18 Feb 2014 11:40:53 GMT
    Connection: close
    Expires: Tue, 18 Feb 2014 14:18:22 GMT
    Cache-Control: max-age=600
    Vary: Accept-Encoding
    Accept-Ranges: bytes


    <!DOCTYPE html>
    <html lang="en">
      <head>
       
      <link rel="icon" type="image/x-icon" href="/favicon.ico" />
        <meta charset="utf-8">
        <meta name="google-site-verification" content="LYB4yqjlrK8xP-NyFUpqzka4SE2PllBFl33Lrcnpx9o" />
        <title>
          
            What a wonderful world ! - 
          
          Coding~
        </title>
        
        <meta name="author" content="Fish">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <!-- HTML5 shim, for IE6-8 support of HTML elements -->
        <!--[if lt IE 9]>
          <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
        <![endif]-->


love67.net返回了我们的 URL 请求, 很简单! 数据接收完后, 就关闭 socket.

##关闭 socket

    s.close()

##回顾

在上面的几个代码中, 我们学习到如何:

* 创建一个 socket
* 连接到远程服务器
* 发送数据
* 收到服务器数据

 当你用浏览器打开 www.love67.net 时，其过程也是一样。包含两种类型，分别是客户端和服务器，客户端连接到服务器并读取数据，服务器使用 Socket 接收进入的连接并提供数据。因此在这里 www.love67.net 是服务器端，而你的浏览器是客户端。



接下来我们开始在服务器端做点编码。



#服务器端编程

服务器端编程主要包括下面几步：

1. 打开 socket
2. 绑定到一个地址和端口
3. 侦听进来的连接
4. 接受连接
5. 读写数据

我们已经学习过如何打开 Socket 了，下面是绑定到指定的地址和端口上。
##绑定 Socket

bind 函数用于将 Socket 绑定到一个特定的地址和端口，它需要一个类似 connect 函数所需的 sockaddr_in 结构体。 

{% highlight python %}
import socket
import sys

HOST = ''	# Symbolic name meaning all available interfaces
PORT = 8888	# Arbitrary non-privileged port

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Socket created'

try:
	s.bind((HOST, PORT))
except socket.error , msg:
	print 'Bind failed. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
	sys.exit()
	
print 'Socket bind complete'
{% endhighlight %}

 绑定完成后，就需要让 Socket 开始侦听连接。很显然，你不能将两个不同的 Socket 绑定到同一个端口之上。
##连接侦听

绑定 Socket 之后就可以开始侦听连接，我们需要将 Socket 变成侦听模式。socket 的 listen 函数用于实现侦听模式：

	s.listen(10)
	print 'Socket now listening'

listen 函数所需的参数成为 backlog，用来控制程序忙时可保持等待状态的连接数。这里我们传递的是 10，意味着如果已经有 10 个连接在等待处理，那么第 11 个连接将会被拒绝。当检查了 socket_accept 后这个会更加清晰。
##接受连接

示例代码：
{% highlight python %}
import socket
import sys
 
HOST = ''   # Symbolic name meaning all available interfaces
PORT = 8888 # Arbitrary non-privileged port
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Socket created'
 
try:
    s.bind((HOST, PORT))
except socket.error , msg:
    print 'Bind failed. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
    sys.exit()
     
print 'Socket bind complete'
 
s.listen(10)
print 'Socket now listening'
 
#wait to accept a connection - blocking call
conn, addr = s.accept()
 
#display client information
print 'Connected with ' + addr[0] + ':' + str(addr[1])
{% endhighlight %}

运行该程序将会显示：

	$ python server.py
	Socket created
	Socket bind complete
	Socket now listening

现在这个程序开始等待连接进入，端口是 8888，请不要关闭这个程序，我们来通过 telnet 程序来进行测试。

打开命令行窗口并输入：

	$ telnet localhost 8888
	 
	It will immediately show
	$ telnet localhost 8888
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	Connection closed by foreign host.

而服务器端窗口显示的是：

	$ python server.py
	Socket created
	Socket bind complete
	Socket now listening
	Connected with 127.0.0.1:59954

我们可看到客户端已经成功连接到服务器。

上面例子我们接收到连接并立即关闭，这样的程序没什么实际的价值，连接建立后一般会有大量的事情需要处理，因此让我们来给客户端做出点回应吧。

sendall 函数可通过 Socket 给客户端发送数据：
{% highlight python %}
import socket
import sys
 
HOST = ''   # Symbolic name meaning all available interfaces
PORT = 8888 # Arbitrary non-privileged port
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Socket created'
 
try:
    s.bind((HOST, PORT))
except socket.error , msg:
    print 'Bind failed. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
    sys.exit()
     
print 'Socket bind complete'
 
s.listen(10)
print 'Socket now listening'
 
#wait to accept a connection - blocking call
conn, addr = s.accept()
 
print 'Connected with ' + addr[0] + ':' + str(addr[1])
 
#now keep talking with the client
data = conn.recv(1024)
conn.sendall(data)
 
conn.close()
s.close()
{% endhighlight %}

继续运行上述代码，然后打开另外一个命令行窗口输入下面命令：

	$ telnet localhost 8888
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	happy
	happy
	Connection closed by foreign host.

可看到客户端接收到来自服务器端的回应内容。

上面的例子还是一样，服务器端回应后就立即退出了。而一些真正的服务器像 www.love67.net 是一直在运行的，时刻接受连接请求。

也就是说服务器端应该一直处于运行状态，否则就不能成为“服务”，因此我们要让服务器端一直运行，最简单的方法就是把 accept 方法放在一个循环内。
一直在运行的服务器

对上述代码稍作改动：
{% highlight python %}
import socket
import sys
 
HOST = ''   # Symbolic name meaning all available interfaces
PORT = 8888 # Arbitrary non-privileged port
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Socket created'
 
try:
    s.bind((HOST, PORT))
except socket.error , msg:
    print 'Bind failed. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
    sys.exit()
     
print 'Socket bind complete'
 
s.listen(10)
print 'Socket now listening'
 
#now keep talking with the client
while 1:
    #wait to accept a connection - blocking call
    conn, addr = s.accept()
    print 'Connected with ' + addr[0] + ':' + str(addr[1])
     
    data = conn.recv(1024)
    reply = 'OK...' + data
    if not data:
        break
     
    conn.sendall(reply)
 
conn.close()
s.close()
{% endhighlight %}

很简单只是加多一个 while 1 语句而已。

继续运行服务器，然后打开另外三个命令行窗口。每个窗口都使用 telnet 命令连接到服务器：

	$ telnet localhost 5000
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	happy
	OK .. happy
	Connection closed by foreign host.


服务器所在的终端窗口显示的是：

	$ python server.py
	Socket created
	Socket bind complete
	Socket now listening
	Connected with 127.0.0.1:60225
	Connected with 127.0.0.1:60237
	Connected with 127.0.0.1:60239


你看服务器再也不退出了，好吧，用 Ctrl+C 关闭服务器，所有的 telnet 终端将会显示 "Connection closed by foreign host."

已经很不错了，但是这样的通讯效率太低了，服务器程序使用循环来接受连接并发送回应，这相当于是一次最多处理一个客户端的请求，而我们要求服务器可同时处理多个请求。
处理多个连接

为了处理多个连接，我们需要一个独立的处理代码在主服务器接收到连接时运行。一种方法是使用线程，服务器接收到连接然后创建一个线程来处理连接收发数据，然后主服务器程序返回去接收新的连接。

下面是我们使用线程来处理连接请求：
{% highlight python %}
import socket
import sys
from thread import *
 
HOST = ''   # Symbolic name meaning all available interfaces
PORT = 8888 # Arbitrary non-privileged port
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print 'Socket created'
 
#Bind socket to local host and port
try:
    s.bind((HOST, PORT))
except socket.error , msg:
    print 'Bind failed. Error Code : ' + str(msg[0]) + ' Message ' + msg[1]
    sys.exit()
     
print 'Socket bind complete'
 
#Start listening on socket
s.listen(10)
print 'Socket now listening'
 
#Function for handling connections. This will be used to create threads
def clientthread(conn):
    #Sending message to connected client
    conn.send('Welcome to the server. Type something and hit enter\n') #send only takes string
     
    #infinite loop so that function do not terminate and thread do not end.
    while True:
         
        #Receiving from client
        data = conn.recv(1024)
        reply = 'OK...' + data
        if not data:
            break
     
        conn.sendall(reply)
     
    #came out of loop
    conn.close()
 
#now keep talking with the client
while 1:
    #wait to accept a connection - blocking call
    conn, addr = s.accept()
    print 'Connected with ' + addr[0] + ':' + str(addr[1])
     
	#start new thread takes 1st argument as a function name to be run, second is the tuple of arguments to the function.
	start_new_thread(clientthread ,(conn,))
s.close()
{% endhighlight %}

行上述服务端程序，然后像之前一样打开三个终端窗口并执行 telent 命令：

	$ telnet localhost 8888
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	Welcome to the server. Type something and hit enter
	hi
	OK...hi
	asd
	OK...asd
	cv
	OK...cv

服务器端所在终端窗口输出信息如下：

	$ python server.py
	Socket created
	Socket bind complete
	Socket now listening
	Connected with 127.0.0.1:60730
	Connected with 127.0.0.1:60731


线程接管了连接并返回相应数据给客户端。

这便是我们所要介绍的服务器端编程。
结论

到这里为止，你已经学习了 Python 的 Socket 基本编程，你可自己动手编写一些例子来强化这些知识。

你可能会遇见一些问题：Bind failed. Error Code : 98 Message Address already in use，碰见这种问题只需要简单更改服务器端口即可。

[英文原文](http://www.binarytides.com/python-socket-programming-turorial)

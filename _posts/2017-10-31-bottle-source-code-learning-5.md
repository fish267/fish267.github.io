---
author: Fish
layout: post
title:  Bottle 源码阅读(五)-- WSGI 服务封装, 热部署原理
categories: python
tags: system_design
---

# 1. WSGI 服务

Python 内置了一个 WSGI 服务 `wsgiref`, 借用 `simple_server.py` 中的例子, 看下简单的使用, :

```python
from wsgiref.simple_server import make_server


def demo_app(environ, start_response):
    from io import StringIO
    stdout = StringIO()
    print("Hello world!", file=stdout)
    print(file=stdout)
    h = sorted(environ.items())
    for k, v in h:
        print(k, '=', repr(v), file=stdout)
    start_response("200 OK", [('Content-Type', 'text/plain; charset=utf-8')])
    return [stdout.getvalue().encode("utf-8")]


def start_wsgi():
    httpd = make_server('', 8000, demo_app)
    sa = httpd.socket.getsockname()
    print("Serving HTTP on", sa[0], "port", sa[1], "...")
    httpd.serve_forever()
```

![](https://gw.alipayobjects.com/zos/rmsportal/cAUDnPGnqpRDFPDLhHzD.png)

`demo_app()` 函数就是符合WSGI标准的一个HTTP处理函数，它接收两个参数：

    environ：一个包含所有HTTP请求信息的dict对象, 从截图可以看出来包含的信息非常多
    start_response：一个发送HTTP响应的函数

<!--more-->

# 2. 代码热部署

Bottle 热部署的思路, 启动两个进程, 其中一个进程中, 开出一个线程检查文件的变动, 另一个进程, 持续的更新临时文件 lockfile 的 modify_time.

进程和线程的文章, 可以参考[一篇文章搞懂Python中的进程和线程
](http://yangcongchufang.com/%E9%AB%98%E7%BA%A7python%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80/python-process-thread.html)

下面分步骤复现一下:

## 2.1 启动两个进程

先看下面代码, 一直循环创建线程:

```python
def hot_deploy_loop(reloader=False, interval=5):
    if reloader:
        print('Executing command:')
        print([sys.executable] + sys.argv)
        try:
            fd, lock_file = tempfile.mkstemp(prefix='six_web.', suffix='.lock')
            os.close(fd)
            while os.path.exists(lock_file):
                args = [sys.executable] + sys.argv
                print('\nCreating process, lock_file:\n %s' % lock_file)
                sleep(interval)
                p = subprocess.Popen(args)
        except KeyboardInterrupt:
            print('Process terminated.')
        finally:
            if os.path.exists(lock_file):
                os.remove(lock_file)

    # 启动 web 服务
    start_wsgi()

hot_deploy_loop(reloader=True, interval=5)
```

输出如下:

```shell
/Users/fish/pycharm_env/bin/python /Users/fish/Documents/GitHub/six-web/six_web/lab.py
Executing command:
['/Users/fish/pycharm_env/bin/python', '/Users/fish/Documents/GitHub/six-web/six_web/lab.py']

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.wbsbtwi4.lock

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.wbsbtwi4.lock
Executing command:
['/Users/fish/pycharm_env/bin/python', '/Users/fish/Documents/GitHub/six-web/six_web/lab.py']

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.koolx88o.lock
Process terminated.
Process terminated.
Versions/3.5/lib/python3.5/http/server.py", line 138, in server_bind
    socketserver.TCPServer.server_bind(self)
    ....
  File "/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/socketserver.py", line 457, in server_bind
    self.socket.bind(self.server_address)
OSError: [Errno 48] Address already in use
    self.socket.bind(self.server_address)
OSError: [Errno 48] Address already in use
```

从结果可以看出, `p = subprocess.Popen(args)` 一直执行文件自身,

    Executing command:
    ['/Users/fish/pycharm_env/bin/python', '/Users/fish/Documents/GitHub/six-web/six_web/lab.py']

防止进程持续创建, bottle 将 `BOTTLE_CHILD` 和 `BOTTLE_LOCKFILE` 放入了 `os.environ` 中, 当成标志来判断是否需要创建新进程. 在创建进程的时候, 将上一个进程的 `env` 传入进来 `p = subprocess.Popen(args, env=env)`.


```python
def hot_deploy_with_flag(reloader=False, interval=5):
    loop_flag = os.environ.get('BOTTLE_CHILD')
    print(loop_flag)
    if reloader and not loop_flag:
        try:
            fd, lock_file = tempfile.mkstemp(prefix='six_web.', suffix='.lock')
            os.close(fd)
            while os.path.exists(lock_file):
                env = os.environ.copy()
                env['BOTTLE_CHILD'] = 'true'
                args = [sys.executable] + sys.argv
                print('\nCreating process, lock_file:\n %s' % lock_file)
                sleep(interval)
                p = subprocess.Popen(args, env=env)
        except KeyboardInterrupt:
            print('Process terminated.')
        finally:
            if os.path.exists(lock_file):
                os.remove(lock_file)

hot_deploy_with_flag(reloader=True, interval=3)
```

输出如下:


```shell
None

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.s766jrnd.lock

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.s766jrnd.lock
true

Creating process, lock_file:
 /var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/six_web.s766jrnd.lock
true
```

上面的代码已经可以实现热部署的雏形, 总共有俩进程, 每隔 interval 时间, 就会执行一次.

我们更希望仅仅在文件变更后重启, 具体实现思路:

<b>依然保持两个进程, 一个进程一直扫描文件变动, 如果有变动就中断当前进程; 另一个进程一直轮询, 如果另一个进程挂了, 就唤起.</b>

## 2.2 一个进程中断, 另一个进程继续轮询

如果文件被修改, 服务端自动重启的原理, 启动 web 服务的进程中断, 然后被另外一个轮询的进程唤起.


```python
def hot_deploy_with_flag(reloader=False, interval=5):
    loop_flag = os.environ.get('BOTTLE_CHILD')
    print('loop_flag: %s' % loop_flag)
    if reloader and not loop_flag:
        try:
            fd, lock_file = tempfile.mkstemp(prefix='six_web.', suffix='.lock')
            os.close(fd)
            while os.path.exists(lock_file):
                env = os.environ.copy()
                env['BOTTLE_CHILD'] = 'true'
                env['LOCK_FILE'] = lock_file
                args = [sys.executable] + sys.argv
                print('\nCreating another process, current pid: %s' % os.getpid())
                sleep(interval)
                p = subprocess.Popen(args, env=env)
        except KeyboardInterrupt:
            print('Process terminated. pid: %s' % os.getpid())
        finally:
            if os.path.exists(lock_file):
                os.remove(lock_file)

    try:
        bg = MockThreadInterruptted(os.environ.get('LOCK_FILE'))
        bg.start()
    except KeyboardInterrupt:
        pass

class MockThreadInterruptted(threading.Thread):
    def __init__(self, lock_file):
        threading.Thread.__init__(self)
        self.lock_file = lock_file

    def run(self):
        print('End process, pid is %s' % os.getpid())
        _thread.interrupt_main()

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.join()
        print('Mock Thread is exciting!')
        return exc_type

    def __enter__(self):
        self.start()

hot_deploy_with_flag(reloader=True, interval=3)
```

上面代码, 输出如下, 注意观察进程 PID:

```shell
/Users/fish/pycharm_env/bin/python /Users/fish/Documents/GitHub/six-web/six_web/lab.py
loop_flag: None

Creating another process, current pid: 15128

Creating another process, current pid: 15128
loop_flag: true
End process, pid is 15130

Creating another process, current pid: 15128
loop_flag: true
End process, pid is 15133

Creating another process, current pid: 15128
loop_flag: true
End process, pid is 15137

Creating another process, current pid: 15128
loop_flag: true
End process, pid is 15138
```

# 3. 文件变更监测

Bottle 监测的是环境变量中的所有文件, `py` `pyc` 结尾的文件, 如果文件的 `modify_time` 发生了变更, 就中断该进程, 中断命令使用 `interrupt_main`, 源码注释如下:

```python
def interrupt_main(): # real signature unknown; restored from __doc__
    """
    interrupt_main()

    Raise a KeyboardInterrupt in the main thread.
    A subthread can use this function to interrupt the main thread.
    """
    pass
```

那么问题转换为, 如何获取当前运行环境中的所有文件, 以及如何获取文件的 `modify_time`. 会使用到 `getattr` 方法和 `st_time`, 分别看一下源码注释:

```python
def getattr(object, name, default=None): # known special case of getattr
    """
    getattr(object, name[, default]) -> value

    Get a named attribute from an object; getattr(x, 'y') is equivalent to x.y.
    When a default argument is given, it is returned when the attribute doesn't
    exist; without it, an exception is raised in that case.
    """
    pass
#
st_mtime = property(lambda self: 0)
"""time of last modification

:type: int
"""
```

简单的实现如下:

```python
def _fetch_all_files():
    for module in sys.modules.values():
        path = (getattr(module, '__file__', ''))
        if path and os.path.exists(path):
            print(path)
            print(os.stat(path).st_mtime)
```

```shell
/Users/fish/pycharm_env/bin/python /Users/fish/Documents/GitHub/six-web/six_web/lab.py
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/datetime.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/encodings/__init__.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/tempfile.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/posixpath.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/signal.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/lib-dynload/_socket.cpython-35m-darwin.so
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/io.py
1449452353.0
/usr/local/Cellar/python3/3.5.1/Frameworks/Python.framework/Versions/3.5/lib/python3.5/contextlib.py
```

## 加上时间判断

```python
def _monitor_file_change(status, lock_file):
    file_history = dict()
    for module in list(sys.modules.values()):
        path = getattr(module, '__file__', '')
        if path and os.path.exists(path):
            ## 将所有文件, 文件时间先存起来
            file_history[path] = os.stat(path).st_mtime
    if not os.path.exists(lock_file):
        _thread.interrupt_main()
    while not status:
        print('Checking file, pid %s' % os.getpid())
        for path, last_modify_time in list(file_history.items()):
            if not os.path.exists(path) or os.stat(path).st_mtime > float(last_modify_time):
                print('File %s has changed!! ' % path)
                status = 'reload'
                _thread.interrupt_main()
                break
        sleep(3)
```

修改当前文件, 输出如下:

```shell
/Users/fish/pycharm_env/bin/python /Users/fish/Documents/GitHub/six-web/six_web/lab.py
Traceback (most recent call last):
  File "/Users/fish/Documents/GitHub/six-web/six_web/lab.py", line 194, in <module>
    # run(reloader=True, port=8000, interval=3)
  File "/Users/fish/Documents/GitHub/six-web/six_web/lab.py", line 187, in _monitor_file_change
    _thread.interrupt_main()
KeyboardInterrupt
File /Users/fish/Documents/GitHub/six-web/six_web/lab.py has changed!

Process finished with exit code 1
```

# 结论

综上所述, 简版的热部署就实现了:

```python
import _thread
import os
import subprocess
import sys
import tempfile
import threading
from time import sleep
from wsgiref.simple_server import make_server


def demo_app(environ, start_response):
    from io import StringIO
    stdout = StringIO()
    print("Hello world!", file=stdout)
    print(file=stdout)
    h = sorted(environ.items())
    for k, v in h:
        print(k, '=', repr(v), file=stdout)
    start_response("200 OK", [('Content-Type', 'text/plain; charset=utf-8')])
    return [stdout.getvalue().encode("utf-8")]


def start_wsgi():
    httpd = make_server('', 8000, demo_app)
    sa = httpd.socket.getsockname()
    print("Serving HTTP on", sa[0], "port", sa[1], "...")
    httpd.serve_forever()


def hot_deploy_with_flag(reloader=False, interval=5):
    loop_flag = os.environ.get('BOTTLE_CHILD')
    print('loop_flag: %s' % loop_flag)
    if reloader and not loop_flag:
        try:
            fd, lock_file = tempfile.mkstemp(prefix='six_web.', suffix='.lock')
            os.close(fd)
            while os.path.exists(lock_file):
                env = os.environ.copy()
                env['BOTTLE_CHILD'] = 'true'
                env['LOCK_FILE'] = lock_file
                args = [sys.executable] + sys.argv
                print('\nCreating another process, current pid: %s' % os.getpid())
                sleep(interval)
                p = subprocess.Popen(args, env=env)
                while p.poll() is None:  # Busy wait...
                    # print('Updating lock_file time %s' % os.getpid())
                    os.utime(lock_file, None)  # I am alive!
                    sleep(interval)
                    # if p.poll() != 3:
                    #     if os.path.exists(lock_file): os.unlink(lock_file)
                    #     sys.exit(p.poll())
        except KeyboardInterrupt:
            print('Process terminated. pid: %s' % os.getpid())
        finally:
            if os.path.exists(lock_file):
                os.remove(lock_file)

    try:
        # lock_file = os.environ.get('LOCK_FILE')
        # bg = FileCheckerThread(lock_file, interval)
        bg = MockThreadInterruptted(os.environ.get('LOCK_FILE'))
        with bg:
            start_wsgi()
    except KeyboardInterrupt:
        pass


class MockThreadInterruptted(threading.Thread):
    def __init__(self, lock_file):
        threading.Thread.__init__(self)
        self.lock_file = lock_file
        self.status = None

    def run(self):
        _monitor_file_change(self.status, self.lock_file)

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.join()
        print('Mock Thread is exciting!')
        return exc_type

    def __enter__(self):
        self.start()


def _monitor_file_change(status, lock_file):
    file_history = dict()
    for module in list(sys.modules.values()):
        path = getattr(module, '__file__', '')
        if path and os.path.exists(path):
            ## 将所有文件, 文件时间先存起来
            file_history[path] = os.stat(path).st_mtime
    if not os.path.exists(lock_file):
        _thread.interrupt_main()
    while not status:
        print('Checking file, pid %s' % os.getpid())
        for path, last_modify_time in list(file_history.items()):
            if not os.path.exists(path) or os.stat(path).st_mtime > float(last_modify_time):
                print('File %s has changed!! ' % path)
                status = 'reload'
                _thread.interrupt_main()
                break
        sleep(3)


hot_deploy_with_flag(reloader=True, interval=3)

# run(reloader=True, port=8000, interval=3)
# _monitor_file_change()
```

文件检查部分, bottle 源码更细致:

```python

class FileCheckerThread(threading.Thread):
    ''' Interrupt main-thread as soon as a changed module file is detected,
        the lockfile gets deleted or gets to old. '''

    def __init__(self, lockfile, interval):
        threading.Thread.__init__(self)
        self.lockfile, self.interval = lockfile, interval
        mtime = lambda path: os.stat(path).st_mtime
        print(mtime(self.lockfile))
        #: Is one of 'reload', 'error' or 'exit'
        self.status = None

    def run(self):
        exists = os.path.exists
        mtime = lambda path: os.stat(path).st_mtime
        files = dict()

        for module in list(sys.modules.values()):
            path = getattr(module, '__file__', '')
            if path[-4:] in ('.pyo', '.pyc'): path = path[:-1]
            if path and exists(path): files[path] = mtime(path)

        while not self.status:
            print('run file checking')
            print('bottle pid: %s' % os.getpid())
            if not exists(self.lockfile) \
                    or mtime(self.lockfile) < time.time() - self.interval - 5:
                print('lockfile is missing')
                self.status = 'error'
                thread.interrupt_main()
            for path, lmtime in list(files.items()):
                if not exists(path) or mtime(path) > lmtime:
                    self.status = 'reload'
                    print('interrupt main thread')
                    thread.interrupt_main()
                    break
            time.sleep(self.interval)

    def __enter__(self):
        self.start()

    def __exit__(self, exc_type, exc_val, exc_tb):
        if not self.status: self.status = 'exit'  # silent exit
        self.join()
        return exc_type is not None and issubclass(exc_type, KeyboardInterrupt)
```


# END
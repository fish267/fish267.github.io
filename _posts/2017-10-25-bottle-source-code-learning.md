---
author: Fish
layout: post
title: Bottle 源码阅读(三) -- ArgumentParser, __slots__, __call__, templefile 模块使用
categories: python
tags: system_design
---


## 1. Python 命令行解析工具

```shell
(pycharm_env) ➜  six_web git:(six_20171018) ✗ ifconfig --help
ifconfig: illegal option -- -
usage: ifconfig [-C] [-L] interface address_family [address [dest_address]]
                [parameters]
       ifconfig interface create
       ifconfig -a [-C] [-L] [-d] [-m] [-u] [-v] [address_family]
       ifconfig -l [-d] [-u] [address_family]
       ifconfig [-C] [-L] [-d] [-m] [-u] [-v]

```

类似在 Linux 系统上执行 `command -help` 命令, C 语言使用的 `getopt()` 进行命令行交互. Python 标准库中自带的 `argparse` 的用法几乎一样.

<!--more-->

```python
    from argparse import ArgumentParser
    parser = ArgumentParser()
    parser.parse_args()
```

输出如下:

```shell
(pycharm_env) ➜  six_web git:(six_20171018) ✗ python six.py --help
usage: six.py [-h]

optional arguments:
  -h, --help  show this help message and exit
```

具体的参数不再赘述, 工程中使用的代码如下:

```python
def _cli_parse(args):
    '''
        python -m six_web 输出的帮助文档
    :param args: 参数输入
    :return:
    '''
    from argparse import ArgumentParser
    parser = ArgumentParser(prog=args[0], usage="%(prog)s [options] package.moudle:app",
                            epilog='Life is short, love python!',
                            description='Coding to change the world for better.')
    opt = parser.add_argument
    opt('-v', '--version', action='store_true', help='show version number')
    opt('-b', '--bind', metavar='ip:port', help='bind address, default localhost:8080')
    opt('-s', '--server', metavar='SERVER', help='user SERVER as backend')
    opt('-C', '--param', metavar='NAME=VALUE', help='overwrite config value')
    opt('--debug', help='run server in debug mode')
    opt('--reload', help='auto reload on file changes')
    parser.parse_args()


if __name__ == '__main__':
    _cli_parse(sys.argv)

```

运行结果:

```shell
(pycharm_env) ➜  six_web git:(six_20171018) ✗ python six.py -h
usage: six.py [options] package.moudle:app

Coding to change the world for better.

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show version number
  -b ip:port, --bind ip:port
                        bind address, default localhost:8080
  -s SERVER, --server SERVER
                        user SERVER as backend
  -C NAME=VALUE, --param NAME=VALUE
                        overwrite config value
  --debug DEBUG         run server in debug mode
  --reload RELOAD       auto reload on file changes

Life is short, love python!
```


## 2. Python __slots__  使用

正常情况下，当我们定义了一个class，创建了一个class的实例后，我们可以给该实例绑定任何属性和方法

```python
>>> class T():
...     pass
...
>>> t = T()
>>> t.name = 'haha'
```

使用 __slots__ 可以限制类的属性与方法, 强行赋值就会报错

```python
>>> class WithSlots():
...     __slots__ = ('name', 'props')
...
>>> s = WithSlots()
>>> s.name = 'haha'
>>> s.name
'haha'
>>> s.props = [i for i in range(100)]
>>> s.pp = 'test'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'WithSlots' object has no attribute 'pp'
```

另外, 动态绑定会消耗不少内存, 使用 __slots__ 后, 只给指定属性分配空间, 很大程度上减少了内存消耗.

[https://eastlakeside.gitbooks.io/interpy-zh/content/slots_magic/](https://eastlakeside.gitbooks.io/interpy-zh/content/slots_magic/)



## 3. Python 临时文件 tempfile

tempfile 模块用于创建临时文件和目录, 用完后自动删除, 比较常用的有 `TemporaryFile` 和 `NamedTemporaryFile`.

比如 Bottle 框架的 config 类测试源码, 其中一个是来测试配置文件读取, 使用方法如下:

```python

class TestINIConfigLoader(unittest.TestCase):
    @classmethod
    def setUpClass(self):
        self.config_file = tempfile.NamedTemporaryFile(suffix='.example.ini',
                                                       delete=True)
        self.config_file.write(b'[DEFAULT]\n'
                               b'default: 45\n'
                               b'[bottle]\n'
                               b'port = 8080\n'
                               b'[ROOT]\n'
                               b'namespace.key = test\n'
                               b'[NameSpace.Section]\n'
                               b'sub.namespace.key = test2\n'
                               b'default = otherDefault\n'
                               b'[compression]\n'
                               b'status=single\n')
        self.config_file.flush()

    @classmethod
    def tearDownClass(self):
        self.config_file.close()

    def test_load_config(self):
        c = ConfigDict()
        c.load_config(self.config_file.name)
        self.assertDictEqual({
            'compression.default': '45',
            'compression.status': 'single',
            'default': '45',
            'namespace.key': 'test',
            'namespace.section.default': 'otherDefault',
            'namespace.section.sub.namespace.key': 'test2',
            'port': '8080'}, c)
```

NamedTemporaryFile, 能获取到文件名.

```python
>>> import tempfile
>>> temp = tempfile.NamedTemporaryFile()
>>> temp.name
'/var/folders/l_/9w7p8m0x2gb5gn36zw8y6wkh0000gn/T/tmpwst_12re'
>>> import os
>>> import os.path
>>> os.path.exists(temp.name)
True
>>> temp.close()
>>> os.path.exists(temp.name)
False
```

源码中还使用了 mkstemp 方法, 返回描述符与文件名, 但是 os.close(fd) 后文件依然存在, 需要手动删除文件

```python
  tempfile.mkstemp([suffix=''[, prefix='tmp'[, dir=None[, text=False]]]])
```

mkstemp方法用于创建一个临时文件。该方法仅仅用于创建临时文件，调用tempfile.mkstemp函数后，返回包含两个元素的元组，
第一个元素指示操作该临时文件的安全级别，第二个元素指示该临时文件的路径。

参数suffix和prefix分别表示临时文件名称的后缀和前缀；

dir指定了临时文件所在的目录，如果没有指定目录，将根据系统环境变量TMPDIR, TEMP或者TMP的设置来保存临时文件


```python
    if reloader and not os.environ.get('BOTTLE_CHILD'):
        import subprocess
        lockfile = None
        try:
            fd, lockfile = tempfile.mkstemp(prefix='bottle.', suffix='.lock')
            os.close(fd)  # We only need this file to exist. We never write to it
            while os.path.exists(lockfile):
                args = [sys.executable] + sys.argv
```

## 4. __call__ 用法

AppStack() 类中, 使用了 __call__ 方法, 

```python
class AppStack(list):
    """ A stack-like list. Calling it returns the head of the stack. """

    def __call__(self):
        """ Return the current default application. """
        return self.default

    def push(self, value=None):
        """ Add a new :class:`Bottle` instance to the stack """
        if not isinstance(value, Bottle):
            value = Bottle()
        self.append(value)
        return value
    new_app = push

    @property
    def default(self):
        try:
            return self[-1]
        except IndexError:
            return self.push()
```

在实例化对象后, 每次调用该对象, 就会执行一次.


```python
>>> class CallClass():
...     def __init__(self, value):
...             self.value = value
...     def __call__(self):
...             self.value = self.value * 10
...
>>> call = CallClass(6)
>>> call.value
6
>>> call
<__main__.CallClass object at 0x10b2ba630>
>>> call.value
6
>>> call()
>>> call.value
60
>>> call.__call__()
>>> call.value
600
```
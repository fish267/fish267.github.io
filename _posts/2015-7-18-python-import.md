---
author: Fish
layout: post
title: Python import探索 
categories: Python 
tags: code
---
Java和Python玩家对import关键字一点都不陌生，本文以python为例，试着理解一下。

##1. 环境变量
可以猜到，在import模块时，python会从环境变量中搜索需要加载的模块，这个列表就存放在sys.path变量中，可以进行修改。想要引入时，首先要将路径放到环境变量中。
对环境变量临时修改，将```/home/admin/git```添加进来，示例如下:
```bash
>>> import sys
>>> '/home/admin/git' in sys.path
False
>>> sys.path.append('/home/admin/git')
>>> '/home/admin/git' in sys.path
True
>>> print sys.path
['', '/usr/lib64/python26.zip', '/usr/lib64/python2.6', '/usr/lib64/python2.6/plat-linux2', '/usr/lib64/python2.6/lib-tk', '/usr/lib64/python2.6/lib-old', '/usr/lib64/python2.6/lib-dynload', '/usr/lib64/python2.6/site-packages', '/usr/lib64/python2.6/site-packages/gtk-2.0', '/usr/lib/python2.6/site-packages', '/home/admin/git']
```
类似windows系统变量，在查询时，也是按照列表的顺序进行遍历
##2. 模块查询和加载
参考[python PEP302][1]，详细讲解了如何导入钩子机制，简单分为了两步模块导入和模块加载

    The protocol involves two objects: a finder and a loader . A finder object has a single method:

    finder.find_module(fullname, path=None)
    loader.load_module(fullname)
###2.1  **finder.find_module**实现
这个很容易理解，文件查询而已，有的是.py文件，有的是带目录package的。

```python
import sys
import os
class ModuleFinder:
    def find_on_path(self, fullname):
        fls = ['%s/__init__.py', '%s.py']
        # import xx.oo.tt package导入，和java导入一样
        dirpath = '/'.join(fullname.split("."))

        for path in sys.path:
            path = os.path.abspath(path)
            for fp in fls:
                composed_path = fp % ("%s/%s" %(path, dirpath))
                if os.path.exists(composed_path):
                    # print composed_path
                    return composed_path

    def find_module(self, fullname, path=None):
        path = self.find_on_path(fullname)
        if path:
            do_something_else()
```
测试代码和结果如下:
```python
# test case
f = ModuleFinder()
f.find_module('os')
f.find_module('urllib')
f.find_module('json')
```
```bash
/usr/lib64/python2.6/os.py
/usr/lib64/python2.6/urllib.py
/usr/lib64/python2.6/json/__init__.py
```                                                                                                                                                                                  
##2.2 loader.load_module的实现
在[PEP302][2]中，给出了一个简单的模块加载器:
```python
# Consider using importlib.util.module_for_loader() to handle
# most of these details for you.
def load_module(self, fullname):
    code = self.get_code(fullname)
    ispkg = self.is_package(fullname)
    mod = sys.modules.setdefault(fullname, imp.new_module(fullname))
    mod.__file__ = "<%s>" % self.__class__.__name__
    mod.__loader__ = self
    if ispkg:
        mod.__path__ = []
        mod.__package__ = fullname
    else:
        mod.__package__ = fullname.rpartition('.')[0]
    exec(code, mod.__dict__)
    return mod
```
将这个函数补充完善一下, 其中```import_file_to_module```是将源码弄成Python模块对象并返回，还有，要将模块的各类属性给塞进去。以```json```为例，他的属性列表(两个'_'开头结尾的）如下:
```bash
>>> import json
>>> dir(json)
['JSONDecoder', 'JSONEncoder', '__all__', '__author__', '__builtins__', '__doc__', '__file__', '__name__', '__package__', '__path__', '__version__', '_default_decoder', '_default_encoder', 'decoder', 'dump', 'dumps', 'encoder', 'load', 'loads', 'scanner']
>>>
```
具体加载器实现如下：
```python
class ModuleLoader(object):
    def __init__(self, path):
        self.path = path
    # package中包含文件__init__.py
    def is_package(self, fullname):
        dirpath = '/'.join(fullname.split("."))
        
        for path in sys.path:
            path = os.path.abspath(path)
            composed_path = '%s%s/__init__.py' %　(path, dirpath)
            if os.path.exists(composed_path):
                return True
            return False
            
    def load_module(self, fullname):
        if fullname in sys.modules:
            return sys.modules(fullname)
        
        if not self.path:
            return
        sys.modules[fullname] = None
        # 读取源文件，编译成python代码，返回一个Python模块对象
        mod = import_file_to_module(fullname, self.path)
        
        ispkg = self.is_package(fullname)
        
        # 将模块的属性写上
        mod.__file__ = self.path
        mod.__loader__ = self
        mod=.__name__ = fullname
        
        # 如果引入的是一个package，还有其他属性
        if ispkg:
            mod.__path__ = []
            mod.__package__ = fullname
        else:
            mod.__package__ = fullname.rpartition('.')[0]
            
        sys.modules[fullname] = mod
        return mod
```

用的最多的东西反而最容易被忽视，比如C语言printf()，C++的cout，Ruby的Require等等。
拿来玩一玩，跟挖宝一样，蛮有意思的~~

## 参考内容
1. [PEP 302是什么？][3]
2. 书籍《Python高手之路》第二章 模块和库
3. [Experience of Python's “PEP-302 New Import Hooks”][4]


  [1]: https://www.python.org/dev/peps/pep-0302/
  [2]: https://www.python.org/dev/peps/pep-0302/
  [3]: https://ruby-china.org/topics/4897
  [4]: http://programmers.stackexchange.com/questions/154247/experience-of-pythons-pep-302-new-import-hooks

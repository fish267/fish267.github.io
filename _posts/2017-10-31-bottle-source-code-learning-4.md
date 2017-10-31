---
author: Fish
layout: post
title: Bottle 源码阅读(四) -- 注解, functools, load 配置
categories: python
tags: system_design
---


## 1. Python 注解使用

高阶函数, 指的是能将函数当成入参来操作, Python 注解也是这么做的, 使用 `@` 符号, 让代码更简洁.

###  示例

```python
def logger(fn):
    def wrapper():
        print('%s starts to executing' % fn.__name__)
        fn()
        print('%s finished executing' % fn.__name__)
    return wrapper


@logger
def hello():
    print('Hello world!')
##
hello starts to executing
Hello world!
hello finished executing
```

如果一个函数上面多个 decorator, 可以这么理解:

```python
@decorator_one
@decorator_two
def func():
    pass
===>
func = decorator(arg1,arg2)(func)
```

### 带一个入参的

```python
def route(url):
    def wrapper(fn):
        print('visiting url: {}'.format(url))
        fn()

    return wrapper


@route('hello/world')
def index():
    print('I am index')
##
visiting url: hello/world
I am index
```

### 带多个入参

函数上有入参, 需要在 decorator 中再嵌套一层, `def inline(text): fn(text)`

```python
def route(**kwargs):
    def wrapper(fn):
        if 'url' in kwargs.keys():
            print('visiting url: {}'.format(kwargs['url']))

        def inline(text):
            fn(text)

        return inline

    return wrapper


@route(url='hello/world')
def index(name):
    print('I am ' + name)


index('world')
###
visiting url: hello/world
I am world
```

## 2. @functools.wraps(func) 作用

<b>wraps()的作用就是把原函数的相关信息代入到新的函数中</b>

如上面的例子, 使用了装饰器后, index 函数的函数名如下:

```python
print(index.__name__)
print(index.__doc__)
##
visiting url: hello/world
inline
None
```

在闭包函数上, 添加 `@functools.wraps(fn)` 就可以复原函数的 `__name__`  `__doc__`.

## 3. load 函数

```python
def load(target, **namespace):
    """ Import a module or fetch an object from a module.

        * ``package.module`` returns `module` as a module object.
        * ``pack.mod:name`` returns the module variable `name` from `pack.mod`.
        * ``pack.mod:func()`` calls `pack.mod.func()` and returns the result.

        The last form accepts not only function calls, but any type of
        expression. Keyword arguments passed to this function are available as
        local variables. Example: ``import_string('re:compile(x)', x='[a-z]')``
    """
    module, target = target.split(":", 1) if ':' in target else (target, None)
    if module not in sys.modules: __import__(module)
    if not target: return sys.modules[module]
    if target.isalnum(): return getattr(sys.modules[module], target)
    package_name = module.split('.')[0]
    namespace[package_name] = sys.modules[package_name]
    return eval('%s.%s' % (module, target), namespace)
```

上面的`load` 函数, 在源码中大量被使用, 具体的作用, 注释讲解的很清楚, 可以返回文件中 `from xx import xx` 的模块, 变量, 函数.

简单测试一下:

```python

def load(target, **namespace):
    module, target = target.split(":", 1) if ':' in target else (target, None)
    if module not in sys.modules: __import__(module)
    if not target: return sys.modules[module]
    if target.isalnum(): return getattr(sys.modules[module], target)
    package_name = module.split('.')[0]
    namespace[package_name] = sys.modules[package_name]
    print('Start to load: \nmodule:%s\npackage_name: %s\nnamespace: %s' % (module, package_name, namespace))
    object = eval('%s.%s' % (module, target), namespace)
    print(object)
    return object


fp = load('six_web.cli:_get_version')
fp()
###
Start to load:
module:six_web.cli
package_name: six_web
namespace: {'six_web': <module 'six_web' from '/Users/fish/Documents/GitHub/six-web/six_web/__init__.py'>}
<function _get_version at 0x10a2bad08>
six_web version: 2.0.
```


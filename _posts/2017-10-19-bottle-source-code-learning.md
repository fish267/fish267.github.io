---
author: Fish
layout: post
title: Bottle 源码阅读(二) -- Python 项目的开源到 Pypi 服务器
categories: python
tags: system_design
---


Pypi 服务, 类似 Java 的 maven 仓库, 用来托管开源的 Python 模块. 发布 Python 模块, 必须要配置好 `setup.py` 等配置文件.

一般待发布的项目目录, 大概有下面文件


```shell
.
|-- LICENSE.txt
|-- MANIFEST.in
|-- README.rst
|-- data
|   `-- data_file
|-- sample
|   |-- __init__.py
|   `-- package_data.dat
|-- sample.egg-info
|   |-- PKG-INFO
|   |-- SOURCES.txt
|   |-- dependency_links.txt
|   |-- entry_points.txt
|   |-- requires.txt
|   `-- top_level.txt
|-- setup.cfg
|-- setup.py
|-- tests
|   |-- __init__.py
|   |-- __init__.pyc
|   |-- __pycache__
|   |   `-- test_simple.cpython-27-PYTEST.pyc
|   `-- test_simple.py
`-- tox.ini
``` 

<!--more-->

## 1. 准备 setup.py

参考官网的 [Writing the Setup Script](https://docs.python.org/3.6/distutils/setupscript.html) 例子. 一个简单的 `setup.py` 文件内容:

```python
#!/usr/bin/env python

from distutils.core import setup

setup(name='Distutils',
      version='1.0',
      description='Python Distribution Utilities',
      author='Greg Ward',
      author_email='gward@python.net',
      url='https://www.python.org/sigs/distutils-sig/',
      packages=['distutils', 'distutils.command'],
     )
```

主要参数说明: 

    name: 包名称, 也就是能够通过 import name 被导入的名字
    packages: 安装的包的路径
    version: 版本号 
    description: PyPI首页的一句话短描述
    long_description: PyPI首页的正文, 通常就是 README.rst 文件中的内容
    url: 你的项目主页, 通常是项目的Github主页
    download_url: 你项目源码的下载链接
    license: 版权协议名


## 2. 准备 setup.cfg  MANIFEST.in

setup.cfg 是一个 ini 格式的文件，可能包含传递给 setup.py 的命令的可选默认值。

MANIFEST.in 此文件中包含有不属于内部包目录，但你仍想纳入进来的文件。这些文件通常是 readme 文件，license 文件以及一些类似的文件。其中，比较重要的一个是 requirements.txt。 pip 使用该文件安装其他必须的包。

```c
include LICENSE
include README.md
include requirements.txt
```


## 3. 本地测试

setup.py文件的使用：

```shell
# 制作分发包
% python setup.py sdist      
# 本地安装测试
% pip install dist/six_web-1.0.0.tar.gz  --upgrade
# 本地卸载
% pip uninstall six_web
```


## 4. 上传到服务器

本地测试完成后, 去 [Pypi官网](https://pypi.python.org/) 注册账号, 然后将 wheel 包上传到服务器, wheel 包的好处是安装的时候, 无需构建, 速度较快, 上传到服务器有两种方式


```shell
# 安装 wheel 工具
pip install wheel
```

此处有个坑, 记得在 setup.py 文件引入 setup 包时, 优先使用 setuptools 中的 setup

```python
try:
    from setuptools import setup
except ImportError:
    from distutils.core import setup
```

### 4.1 使用 twine 上传

[twine](https://pypi.python.org/pypi/twine)是一个专门用于发布项目到PyPI的工具，可以使用 `pip install twine` 来安装.

```shell
# 生成 wheel 安装包
python setup.py bdist_wheel

# 上传安装包
twine upload  -u username -p password dist/six_web-1.0.0-py3-none-any.whl
```

可以在服务器上直接看到上传的包了

![](https://gw.alipayobjects.com/zos/rmsportal/xcaOSwPjUTcCUKaMbjDE.png)

# 参考资料

1. [https://packaging.python.org/tutorials/distributing-packages/](https://packaging.python.org/tutorials/distributing-packages/)
2. [https://github.com/pypa/sampleproject](https://github.com/pypa/sampleproject)
3. [https://stackoverflow.com/questions/26664102/why-can-i-not-create-a-wheel-in-python](https://stackoverflow.com/questions/26664102/why-can-i-not-create-a-wheel-in-python)
4. [https://github.com/fish267/six-web](https://github.com/fish267/six-web)
---
author: Fish
layout: post
title: Bottle 源码阅读(一) -- 使用 Travis CI 及 Coveralls 持续集成
categories: python
tags: system_design 
---

Clone 下来 Bottle 源码后, 逐个文件分析, 发现 `.travis.yml` 文件中的配置比较有意思, 如下图. Google 了一下, 发现是 Github 项目持续集成用的.

```yml
language: python
sudo: required

python:
  - "2.7.3" # Ubuntu 12.4LTS (precise) and Debian 7 LTS (wheezy)
  - "2.7"
  - "3.3"
  - "3.4"
  - "3.5"
  - "3.6"
  - "nightly"

install:
  - travis_retry bash test/travis_setup.sh

script:
  - python -m coverage run --source=. test/testall.py fast
  - python -m coverage combine
  - python -m coverage report 2>&1

notifications:
  irc: "irc.freenode.org#bottlepy"
  on_success: "never"

after_success:
  coveralls
```

## 1. 什么是 Travis 和 Coveralls
Github 项目经常会看到类似  ![](https://travis-ci.org/bottlepy/bottle.svg?branch=master)  ![]( https://coveralls.io/repos/github/bottlepy/bottle/badge.svg?branch=master) 图标, 分别对应了 Travis 和 Coveralls.

Travis-CI是一个开源的持续构建项目，同步你在GitHub上托管的项目，每当你Commit Push之后, 就会在几分钟内开始按照你的要求测试部署你的项目, 并跑一遍测试用例.

Coveralls 是用来显示项目代码覆盖率的.

<!--more-->

## 2. Python 项目测试工程配置

每次初始化一个新项目, 优先构建好测试配置, 是个非常好的习惯.

比如新建一个项目, 取名 **six**, 目录结构大致如下, 新建了 <code>test</code> <code>test/testcase</code> 两个目录.
```shell
└── six
    ├── __pycache__
    │   └── six.cpython-35.pyc
    ├── six.py
    └── test
        ├── __pycache__
        ├── test_all.py
        ├── testcase
           ├── __pycache__
           └── test_demo.py
```

新建 <code>test_all.py</code> 文件夹, 用来执行所有的测试用例, 需要获取项目覆盖率, 依赖 coverage, 使用

 `pip install -U coverage` 进行安装. 文件内容如下:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import sys
import unittest

# Add test directory to sys.path
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

try:
    import coverage
    coverage.process_startup()
except ImportError:
    pass


def generatte_test_suit():
    '''
        Add testcases into test suite
    :return:
    '''
    current_pwd = os.path.dirname(__file__)
    testcase_directory = current_pwd + os.sep + 'testcase'
    return unittest.defaultTestLoader.discover(testcase_directory)


if __name__ == '__main__':
    suite = generatte_test_suit()
    unittest.TextTestRunner().run(suite)
```

上面代码逻辑大概是定位到 <code>testcase</code> 目录,  `unittest.defaultTestLoader.discover` 找到并加装所有 `test*.py` 的文件, 最后调用 `unittest.TextTestRunner().run` 方法执行.

`sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))`

这句话非常重要, 将 `test_all.py` 文件的上一级目录加载到 `sys.path` 中, 不然会报类找不到的.

**在项目根目录, 执行下面命令, 将用例跑一遍:**


```python
python -m coverage run  test/test_all.py

# 结果
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

```

**查看测试覆盖率, 执行下面命令**

```python
python -m coverage report -m

# 结果
Name                         Stmts   Miss  Cover   Missing
----------------------------------------------------------
six.py                           4      0   100%
test/test_all.py                16      2    88%   15-16
test/testcase/test_demo.py       6      0   100%
----------------------------------------------------------
TOTAL                           26      2    92%
```

## 3. Travis 配置

有了第二部分的操作, 就可以配置 `.travis.yml` 了

### 3.1 Travis 设置

访问 [https://travis-ci.org/](https://travis-ci.org/), 使用 Github 账号授权登录, 将需要持续集成的仓库状态打开.

![](https://gw.alipayobjects.com/zos/rmsportal/ZwabjnvEcWALXlRRYMud.png)

### 3.2 项目 .travis.yml 配置

针对不同编程语言, travis 官网文档提供了不同的配置 demo

[https://docs.travis-ci.com/user/language-specific/](https://docs.travis-ci.com/user/language-specific/)

Python 项目的配置如下:

```yml
language: python
python:
  - 3.6
  - nightly
install:
  - sh ./test/travis_config/travis_setup.sh
script:
  - sh ./test/travis_config/travis_run.sh

after_success:
  coveralls
```

`python` 对应的需要回归的 Python 版本, `install` 是准备脚本, 一般执行一些安装命令, `script` 就是执行脚本, 可以不需要写到 shell 脚本中, 直接写到配置文件中.

`travis_setup.sh` 文件内容:

```shell
#!/usr/bin/env bash

# Just to be sure
pip install -U pip
# pip is not able to install distribute: "ImportError: No module named _markerlib"
easy_install distribute

pip install -U coverage
pip install coveralls
```

`travis_run.sh` 文件内容:

```shell
#!/usr/bin/env bash

python -m coverage run  test/test_all.py
python -m coverage combine
python -m coverage report -m 2>&1
```

上述配置可以在项目[https://github.com/fish267/six-web](https://github.com/fish267/six-web) 中查看.

提交代码后, 过一会儿就能收到持续集成的邮件, 也可以访问 [https://travis-ci.org/fish267/six](https://travis-ci.org/fish267/six) 查看构建历史.

![](https://gw.alipayobjects.com/zos/rmsportal/dOCZhsYQBEnwhUBCWGLg.png)

## 4. Coveralls 配置

通上配置, 在官网 [https://coveralls.io](https://coveralls.io) 使用 Github 账号授权登录, 打开 Repo 的开关.


![](https://gw.alipayobjects.com/zos/rmsportal/EeMwqQaCgnbADoXkULem.png)

### 4.1 .travis.yml 配置

在 `.travis.yml` 文件添加如下配置, 就能直接看到覆盖率的条条了. 如 [https://coveralls.io/github/fish267/six](https://coveralls.io/github/fish267/six)

```yml
after_success:
  coveralls
```

![](https://gw.alipayobjects.com/zos/rmsportal/seMPIwqRQdEVVyCocXxe.png)


## 参考资料

0. [https://docs.travis-ci.com/](https://docs.travis-ci.com/)
2. [http://docs.coveralls.io/](http://docs.coveralls.io)
3. [https://coverage.readthedocs.io/en/coverage-4.4.1/](https://coverage.readthedocs.io/en/coverage-4.4.1/)
4. [用正确的姿势开源Python项目](http://moelove.info/2015/10/26/%E7%94%A8%E6%AD%A3%E7%A1%AE%E7%9A%84%E5%A7%BF%E5%8A%BF%E5%BC%80%E6%BA%90Python%E9%A1%B9%E7%9B%AE/)
5. [如何简单入门使用Travis-CI持续集成](https://github.com/nukc/how-to-use-travis-ci)


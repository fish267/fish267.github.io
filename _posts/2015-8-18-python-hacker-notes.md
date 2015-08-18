---
author: Fish
layout: post
title: 《Python 高手之路》读书笔记 <一>
categories: Python 
tags: Linux 
---

 《Python 高手之路》这本书很不错，翻译的也部分到位！先贴一段介绍：

     Python 是一门优美的语言， 它快速，灵活且内置了丰富的标准库， 已经用于越来越多的不同领域。 通常大多数关于 Python 的书都会教读者这门语言的基础知识， 但是掌握了这些基础知识后， 读者在设计自己的应用程序和探索最佳实践时仍需要完全靠自己。 本书则不同， 介绍了如何利用 python 有效的解决问题， 以及如何构建良好的 python 应用程序。

# 项目开始

## 1. python 项目结构

![此处输入图片的描述][1]

上图展示了一个项目的标准目录层次结构。

## 2. python 版本编号

[PEP440][2] 中规定了 python 的版本号应当遵照以下正则表达式：

     [N!]N(.N)*[{a|b|rc}N][.postN][.devN]
     
1.2a1 表示 alpha 版本，不是很稳定

2.3.1b2 一个beta版本，差不多了，有点Bug

## 3. 代码风格与编码规范

Python! 毫无疑问，<b>PEP8</b>， 参考地址 [PEP 0008 -- Style Guide for Python Code][3]

还有个译文版！ 真赞 ！！  [<译> PEP 8 - Python 编码风格指南][4]
<!--more-->
具体如下：

* Tab 使用4个空格
* 每行最多79个字符
* 顶层的函数或者类的定义之间空两行
* 采用 ASCII 或者 UTF-8 编码文件
* 在文件顶端，注释和说明文档下，每行每条 import 语句只导入一个模块， 同事按照标准库，第三方库和本地库的导入顺序进行分组
* 在小括号，中括号，大括号和逗号之间或者之前，没有空格
* 类的命名使用驼峰命名，如 CamelWind, 函数的命名使用小写字母加下划线， camel_wind_function; 用下划线开头定义私有的属性或者方法，如 _private

说了这么多， 用 pep8 这个工具就行。

{% highlight shell %}
root@bimeizi:~/meizi.com# pip install pep8
Downloading/unpacking pep8
  Downloading pep8-1.6.2-py2.py3-none-any.whl (40kB): 40kB downloaded
Installing collected packages: pep8
Successfully installed pep8
Cleaning up...
root@bimeizi:~/meizi.com# pep8 meizi.py
meizi.py:7:1: E265 block comment should start with '# '
meizi.py:12:1: E265 block comment should start with '# '
meizi.py:16:1: E265 block comment should start with '# '
meizi.py:18:13: E251 unexpected spaces around keyword / parameter equals
meizi.py:18:15: E251 unexpected spaces around keyword / parameter equals
{% endhighlight %}

# 模块、系统、文档

## 1. import 分析

参考这个文章 [Python import探索][5]

## 2. 文档
个人感觉 python 文档相对顺眼很多， 得益于一些工具， 使得写文档和写代码一样爽~

### 一个项目的文档差不多包括以下内容

* 用一两句话描述项目解决的问题
* 项目基于的分发许可 （开源许可协议等）
* Demo
* 安装指南
* 一些链接（Github, Google Email, Wiki, Bug 等）

## 3. Read The Docs
用这个生成的文档，颜值非常高！比如
 [金融云产品技术文档目录][6]

这个 [http://savu.readthedocs.org/en/latest/user/?highlight=pip][7]

[Read The Docs][8]  可以自动在线生成和发布文档， 可以搜索 Sphinx 配置文件， 构建文档， 以后写开源项目， 简直是 Github 的完美搭配~

# 软件发布与虚拟环境

## 1. 软件发布

to be continued...

## 2. 虚拟环境
**解决的问题：**
1. 系统中没有需要的库
2. 系统中没有库的正确版本
3. 对两个不同的应用程序， 可能需要同一个库的两个不同版本

**使用工具： virtualenv**

这边博客介绍的比书上要简洁： [使用virtualenv搭建独立的Python环境][9]

个人觉得，tox比较叼，抽空仔细分析一下。


  [1]: https://t.alipayobjects.com/images/rmsweb/T10MFhXfFbXXXXXXXX.JPG
  [2]: https://www.python.org/dev/peps/pep-0440/
  [3]: https://www.python.org/dev/peps/pep-000840/
  [4]: http://damnever.github.io/2015/04/24/PEP8-style-guide-for-python-code/
  [5]: http://love67.net/2015/07/18/python-import/
  [6]: http://doc.alipay.net/middleware/index.html
  [7]: http://savu.readthedocs.org/en/latest/user/?highlight=pip
  [8]: http://readthedocs.orgeadthedocs.org/en/latest/user/?highlight=pip
  [9]: http://qicheng0211.blog.51cto.com/3958621/1561685test/user/?highlight=pip

---
author: Fish
layout: post
title: Python 开发环境初始化配置
categories: python
tags: linux
---

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/1a3a8aa2821fbb5755599f8001264ce3/image.png)

本文介绍在 Linux环境下, Python 环境开发需要配置的内容, 包括 python3 安装, virtualenv 配置, pip 等.

### 1. 源码安装 python3

1. 首先去官网找到 source 地址, [Python Source Releases](https://www.python.org/downloads/source/), 以当前最新版本 <code>Python3.6.1</code> 为例, 打开该版本地址 [Python 3.6.1](https://www.python.org/downloads/release/python-361/)

2. 选择第一个源码包, 在 Linux 系统上使用 wget 下载.

		wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz
![e476766683cf5d13c0d14b77d3462fd2](https://private-alipayobjects.alipay.com/alipay-rmsdeploy-image/skylark/attach/1563/e476766683cf5d13c0d14b77d3462fd2)

3. 解压缩, 编译安装 Python3, 安装需要用到 sudo 权限, 提前切换成有 sudo 权限的用户.

```shell
tar -zxvf Python-3.6.1.tgz
cd cd Python-3.6.1
./configure && make && sudo make install
```

安装完成后, terminal 中输入 python3 就能打开解释器
```shell
$python3
Python 3.6.1 (default, Apr  6 2017, 20:09:12)
[GCC 4.4.6 20110731 (Red Hat 4.4.6-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

<!--more-->
### 2. 安装包管理工具 pip

#### 2.1 pip 介绍
pip 之于 Python, 好比 mvn 之于 Java, npm 之于 NodeJS.

好消息是, 如果你使用最新的 Python3 或者 Python2(Python 2 >=2.7.9 or Python 3 >=3.4), 系统默认已经安装了 pip. 

比如上面安装的 Python3, 我们很快找到 pip 的路径:

```shell
$which python3
/usr/local/bin/python3

cd /usr/local/bin

$ls pip*
pip3  pip3.6
```

####  2.2  pip 手动安装

为了兼容 Python 老版本, 介绍下安装 pip 的步骤, 同样需要使用 sudo 权限:

```shell
wget https://bootstrap.pypa.io/get-pip.py && sudo python get-pip.py
```

### 3. 配置 Python 执行环境 virtualenv

在开发Python应用程序的时候, 如果不同项目使用的 Django 版本不同, 第三方的包都会被pip安装到的site-packages目录, 会导致相互覆盖, 因此用到 Python 环境管理器 virtualenv 解决.

virtualenv 主要解决如下问题:

1. 不同项目使用不同版本的包, 相互独立
2. 安装包, 不需要 sudo 权限

#### 3.1 区分 pyenv pyvenv virtualenv



首先理解 3 个容易混淆的概念:

1. <code>pyenv</code> 这是一个python 版本安装器, , 详情见[官网说明]((https://github.com/pyenv/pyenv)). 比如本文 Python3.6.1 可以使用该工具进行安装:

	```
	pyenv install 3.6.1
	```

2. pyvenv

	Python3.3 集成到标准库中的虚拟环境管理器, 放目的是将来替代 virtualenv (参考 [PEP405](https://www.python.org/dev/peps/pep-0405/)), 但是不支持低版本的 Python, 当前功能不是非常完善. 

3. virtualenv

	本地新建目录, 将系统 Python 环境完整拷贝过来, 保持每个 python 环境的独立性.

#### 3.2 virtualenv 使用介绍

- 首先, 安装 virtualenv
```shell
sudo pip3 install virtualenv
```

- 然后, 创建一个独立的 Python 环境, 给项目使用, 比如命名 great_python3_env
```python
virtualenv great_python3_env

Using base prefix '/usr/local'
New python executable in /home/admin/git/great_python3_env/bin/python3.6
Also creating executable in /home/admin/git/great_python3_env/bin/python
Installing setuptools, pip, wheel...done.
```
如果不需要当前环境的包, 使用 ```virtualenv nowamagic_venv --no-site-packages``` 进行安装

- 最后, 激活该环境, 也可以将如下命令添加到 <code>~/.bashrc</code> 或者 <code>~/.zshrc</code> 中
```shell
source great_python3_env/bin/activate

(great_python3_env)
```

这样你就拥有了项目独立占用的 Python 环境, 由于刚才是 pip3 安装的 virtualenv, 因此生成的环境也是基于 Python3 的. 命令行测试如下:

```shell
$which python
~/git/great_python3_env/bin/python

```
可见 python 命令已经指向到了该环境中的 python, 同样, 该环境下使用 pip 命令安装, 也会安装到 <code>~/git/great_python3_env/lib/python3.6/site-packages/</code> 中.

#### 3.3 退出 virtualenv 环境

使用 deactivate 退出当前环境
```shell
deactivate
```  
## 4. 总结
以上就是 Python 开发前的环境准备工作了, 简单整理如下:
```python
# 1. 源码安装
mkdir ~/git && cd git
wget wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz
tar -zxvf Python-3.6.1.tgz
cd Python-3.6.1
./configure && make && sudo make install

# 2. 手动安装 pip
wget https://bootstrap.pypa.io/get-pip.py && sudo python get-pip.py

# 3. 初始化 virtualenv 环境
sudo pip3 install virtualenv
virtualenv great_python3_env
source great_python3_env/bin/activate

# 4. 添加到 .bashrc
echo 'source ~/git/great_python3_env/bin/activate' >>  ~/.bashrc

# 5. 退出 virtualenv
deactivate
```

## 5. 参考文档
1. [Pic 原地址](https://unsplash.com/search/key?photo=feXpdV001o4)
2. [pyenv Github](https://github.com/pyenv/pyenv)
3. [python-pyenv-pyvenv-virtualenv-whats-the-difference](http://masnun.com/2016/04/10/python-pyenv-pyvenv-virtualenv-whats-the-difference.html)
4. [VENV 文档](https://docs.python.org/3/library/venv.html)
5. [PEP405](https://www.python.org/dev/peps/pep-0405/)
6. [pyvenv_vs_virtualenv](https://www.reddit.com/r/learnpython/comments/4hsudz/pyvenv_vs_virtualenv/)

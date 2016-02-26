---
author: Fish
layout: post
title: Python 爬虫解析器之 Beautiful Soup
categories:  python
tags: crawl
---

[Beautiful Soup][1] 是 html/xml 的 python 解析器, 可以定位 DOM 元素信息. 

该工具在[爬虫阶段](http://www.atatech.org/articles/48415) 中, 处于<b>解析网页</b>阶段, 可以简化正则表达式, 简单示例:

```shell
>>> from bs4 import BeautifulSoup as BS
>>> from urllib.request import urlopen
>>> soup = BS(urlopen('https://www.alipay.com/'))
>>> soup.title
<title>支付宝 知托付！</title>
```

## 1. 安装

### pip 安装

```shell
pip install beautifulsoup4
```

<!--more-->
### 源码安装

如果没有 <code>easy_install</code> 和 <code>pip</code>, 需要[下载源码](http://www.crummy.com/software/BeautifulSoup/bs4/download/4.0/), 使用 <code>setup.py</code> 来安装. 


```python
python3 setup.py install
```
## 2. 使用

将网页源码传入 <code>BeautifulSoup</code>的构造函数, 就能使用该工具的各类方法, 如下:
```python
from bs4 import BeautifulSoup as BS

soup = BS('<html><p>Google</p></html>')
print(soup.p.text)
# Google
```
### 2.1 解析Tag

BeautifulSoup 将 html 代码解析成树状结构, 每个节点都是 Tag 对象, 具有 name 以及 class 等属性.

尖括号围起来的就是 Tag, 比如 <code>div a p b meta head body</code>

```python
soup = BS("<div id='soup'><p class='super'>Hello Google!</p></div>")

print(type(soup.p))
# <class 'bs4.element.Tag'>
print(soup.p)
# <p class="super">Hello Google!</p>
print(soup.p.name)
# 'p'
print(soup.div)
# <div id="soup"><p class="super">Hello Google!</p></div>

# 每个 Tag 有部分属性, 比如 class stype type 等, tag['attribute']来调用
print(soup.p['class'])
# ['super']
print(soup.div['id'])
# 'soup'
```
### 2.2 find_all 过滤器

<code>soup.a</code>只能获取到文档树第一个超链接 tag, 如果需要检索出同类型的 tag, 使用 <code>find_all</code> 方法搞定. 

<code>find_all</code> 作为 BeautifulSoup 的过滤器, 是使用最为频繁的方法, 以至于该方法都不用写了, <code>soup.find_all("div")</code>  和 <code>soup("div")</code> 作用一样一样的. 下面仔细说明一下该方法的用法:
```python
soup = BS('''<html><a href='alipay.com'>alipay</a> <div> <a href='google.com'>google</a></div></html>''', 'html.parser')
print(soup.find_all('a'))

# [<a href="alipay.com">alipay</a>, <a href="google.com">google</a>]
```

#### 2.2 过滤 Tag
```python
find_all( name , attrs , recursive , text , **kwargs )
```
传入字符串或者字符串数组, 作为 Tag 的名字来搜索该 Tag

```python
soup = BS('''<html><a href='alipay.com'>alipay</a>
                <div> <a href='google.com'>google</a></div>
                <b class='xx'>yahoo</b>
                <h2 id='h2'>shadowsocks</h2>
             </html>''',
          'html.parser')
```

入参为 tag.name
```python
soup.find_all('a')
# [<a href="alipay.com">alipay</a>, <a href="google.com">google</a>]
```

 入参为字符串数组

```python
print(soup.find_all(['a', 'div']))
# [<a href="alipay.com">alipay</a>, <div> <a href="google.com">google</a></div>, <a href="google.com">google</a>]
```
入参 (tag.name, class)
```python
# 查找 class 为 xx, 类型为 <b> 的 Tag
print(soup.find_all('b', 'xx')
# [<b class="xx">yahoo</b>]
```
根据 id 查找

id 可以替换成 href, style,  text, class_(注意有下划线, 防止和关键字冲突)
```python
print(soup.find(id='hh'))
# [<h2 id='h2'>shadowsocks</h2>]
```

支持正则

如果传入正则表达式作为参数,Beautiful Soup会通过正则表达式的 match() 来匹配内容, 简直爽爆了

```python
import re

for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b

for tag in soup.find_all(re.compile("t")):
    print(tag.name)
# html
# title
```

再高级一点, 入参可以直接传入个函数, 不深究了.

### 2.3 CSS 选择器 select

BeautifulSoup 还支持 [CSS 选择器](http://www.w3school.com.cn/css/css_selector_type.asp), 对于 CSS 玩的溜的人很容易掌握元素定位.

简单举几个例子, 具体使用, 还是以 <code>find_all</code> 为主. 

```python

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

soup = BS(html_doc, 'html.parser')
print(soup.select('title'))
# [<title>The Dormouse's story</title>]

```

根据子标签嵌套查找, 使用 tag.name > tag1.name > tag2.name, 注意留着空格

```python
print(soup.select('body > p > b'))
# [<b>The Dormouse's story</b>]
```

根据 CSS 类名查找

```python

print(soup.select('sister'))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

根据 id 来过滤

```python
print(soup.select('#link1'))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

## 参考资料

1. [官方文档](http://www.crummy.com/software/BeautifulSoup/bs4/doc/index.html)


2. [简单 Python 爬虫](http://www.atatech.org/articles/48415) 


  [1]: https://en.wikipedia.org/wiki/Beautiful_Soup

## END

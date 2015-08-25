---
author: Fish
layout: post
title: 用 Python 玩测试
categories: Linux
tags: python
---
Python 库这么多，测试库肯定也是丰富无比。官网给出了Python测试框架的列表 [PythonTestingToolsTaxonomy][1] ，
常见的比如 <code>unittest nose mock Selenium</code> 等。本文简单了解一下 Python 的测试~

# 单元测试 unittest
首先，给出， unittest 文档地址 [https://docs.python.org/4.5/library/unittest.html][2]，  具体用法，就不翻译了，查查文档就行。

第一句话就乐了， unittest 是借鉴 Junit 实现的。

    The unittest unit testing framework was originally inspired by JUnit and has a similar flavor as major unit testing frameworks in other languages.
    
unittest 支持以下四种方式：

- test fixture 
- test case
- test suite
- test runner

![关系][3]

首先是要写好TestCase，然后由TestLoader加载TestCase到TestSuite，然后由TextTestRunner来运行TestSuite，运行的结果保存在TextTestResult中，整个过程集成在unittest.main模块中。

## unittest Demo

{% highlight python %}
import unittest

class TestStringMethods(unittest.TestCase):

  def test_upper(self):
      self.assertEqual('foo'.upper(), 'FOO')

  def test_isupper(self):
      self.assertTrue('FOO'.isupper())
      self.assertFalse('Foo'.isupper())

  def test_split(self):
      s = 'hello world'
      self.assertEqual(s.split(), ['hello', 'world'])
      # check that s.split fails when the separator is not a string
      with self.assertRaises(TypeError):
          s.split(2)
  @unittest.skip("demonstrating skipping")
  def test_nothing(self):
      self.fail("shouldn't happen")

  @unittest.skipIf(mylib.__version__ < (1, 3),
                   "not supported in this library version")
  def test_format(self):
      # Tests that work for only a certain version of the library.
      pass

  @unittest.skipUnless(sys.platform.startswith("win"), "requires Windows")
  def test_windows_support(self):
      # windows specific testing code
      pass
        
if __name__ == '__main__':
    unittest.main()
{% endhighlight %}




##参考文档

[1. Python单元测试——深入理解unittest][4]


  [1]: https://wiki.python.org/moin/PythonTestingToolsTaxonomy
  [2]: https://docs.python.org/3.5/library/unittest.html
  [3]: http://images.cnitblog.com/i/236038/201404/230040440455234.png
  [4]: http://www.cnblogs.com/hackerain/p/3682019.html

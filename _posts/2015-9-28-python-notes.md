---
author: Fish
layout: post
title: Python Notes
categories: Linux
tags: python
---
本文内容来自，慕课网廖老师 Python 课程，地址： http://www.imooc.com/learn/317

#高阶函数

将函数当做入参：
{% highlight python %}
def half(c): return c / 2.0
def test(a, b, c):
    return c(a) + c(b)
print(test(2, 8, half))  
{% endhighlight %}

###Map


map()是 Python 内置的高阶函数，它接收一个函数 f 和一个 list，并通过把函数 f 依次作用在 list 的每个元素上，得到一个新的 list 并返回。

<!--more-->

{% highlight python %}

def f(x): return x ** 2 
print map(f, [1, 2, 3])

{% endhighlight %}

###Reduce

reduce()函数也是Python内置的一个高阶函数。reduce()函数接收的参数和 map()类似，一个函数 f，一个list，但行为和 map()不同，reduce()传入的函数 f 必须接收两个参数，reduce()对list的每个元素反复调用函数f，并返回最终结果值。

{% highlight python %}
def r(x, y): return x * y
print reduce(r, [1, 2, 3])
{% endhighlight %}

###Filter

filter()函数是 Python 内置的另一个有用的高阶函数，filter()函数接收一个函数 f 和一个list，这个函数 f 的作用是对每个元素进行判断，返回 True或 False，filter()根据判断结果自动过滤掉不符合条件的元素，返回由符合条件元素组成的新list。

{% highlight python %}

def f(x): return x % 2 == 0
print filter(f, range(1, 10))
{% endhighlight %}

###返回函数

{% highlight python %}
def calc_prod(lst):
    def g():
        return reduce(c, lst)
    return g
def c(x, y): return x * y
f = calc_prod([1, 2, 3, 4])
print f()

{% endhighlight %}

函数在 f = ** 时不会执行， f() 才会真正进行内部的运算，起到延时的效果

例如，定义一个函数 f()，我们让它返回一个函数 g，可以这样写：
{% highlight python %}
def f():
    print 'call f()...'
    # 定义函数g:
    def g():
        print 'call g()...'
    # 返回函数g:
    return g
{% endhighlight %}
    
    >>> x = f()   # 调用f()
    call f()...
    >>> x   # 变量x是f()返回的函数：
    <function g at 0x1037bf320>
    >>> x()   # x指向函数，因此可以调用
    call g()...   # 调用x()就是执行g()函数定义的代码

###闭包
内层函数引用了外层函数的变量（参数也算变量），然后返回内层函数的情况，称为闭包（Closure）。

闭包的特点是返回的函数还引用了外层函数的局部变量，所以，要正确使用闭包，就要确保引用的局部变量在函数返回后不能变。


{% highlight python %}

def count():
    fs = []
    for i in range(1, 4):
        def g(k):
            def s():
                return k * k
            return s
        fs.append(g(i))
    return fs

f1, f2, f3 = count()
print f1(), f2(), f3()

{% endhighlight %}

### 匿名函数

关键字lambda 表示匿名函数，冒号前面的 x 表示函数参数。

匿名函数有个限制，就是只能有一个表达式，不写return，返回值就是该表达式的结果。

使用匿名函数，可以不必定义函数名，直接创建一个函数对象，很多时候可以简化代码：

    >>> sorted([1, 3, 9, 5, 0], lambda x,y: -cmp(x,y))
    [9, 5, 3, 1, 0]

### decorator
Python的 decorator 本质上就是一个高阶函数，它接收一个函数作为参数，然后，返回一个新函数。

使用 decorator 用Python提供的 @ 语法，这样可以避免手动编写 f = decorate(f) 这样的代码。

考察一个@log的定义：

{% highlight python %}
def log(f):
    def fn(x):
        print 'call ' + f.__name__ + '()...'
        return f(x)
    return fn
{% endhighlight %}
对于阶乘函数，@log工作得很好：

{% highlight python %}
@log
def factorial(n):
    return reduce(lambda x,y: x*y, range(1, n+1))
print factorial(10)

{% endhighlight %}

查看下面代码

{% highlight python %}
import time


def performance(unit):
    def locator(f):
        def wrapper(*arg, **args):
            start_time = time.time()
            r = f(*arg, **args)
            end_time = time.time()
            print 'call ' + f.__name__ + '() ' + str(end_time - start_time) + 'ms'
            return r

        return wrapper

    return locator


@performance('ms')
def factorial(n):
    print reduce(lambda x, y: x * y, range(1, n + 1))


factorial(10)
{% endhighlight %}


###偏函数

functools.partial就是帮助我们创建一个偏函数的，不需要我们自己定义int2()，可以直接使用下面的代码创建一个新的函数int2：

    >>> import functools
    >>> int2 = functools.partial(int, base=2)
    >>> int2('1000000')
    64
    >>> int2('1010101')
    85
### 类的继承

主要记住构造函数的书写

{% highlight python %}

class Person(object):
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender

class Teacher(Person):

    def __init__(self, name, gender, course):
        super(Teacher, self).__init__(name, gender)
        self.course = course

t = Teacher('Alice', 'Female', 'English')
print t.name
print t.course


{% endhighlight %}

另外一种构造函数

{% highlight python %}

class Person(object):

    def __init__(self, name, gender, **kw):
        self.name = name
        self.gender = gender
        for k, v in kw.iteritems():
            setattr(self, k, v)

p = Person('Bob', 'Male', age=18, course='Python')
print p.age
print p.course
{% endhighlight %}

### __str__ __repr__
因为 Python 定义了__str__()和__repr__()两种方法，__str__()用于显示给用户，而__repr__()用于显示给开发人员。

定义了 __str__，可以 print s 输出自定义的内容。

定义了 __repr__, 执行 >>>s  一样的效果

###@property
@property 就是 getter
@**.setter 就是 setter

### __slots__
由于Python是动态语言，任何实例在运行期都可以动态地添加属性。

如果要限制添加的属性，例如，Student类只允许添加 name、gender和score 这3个属性，就可以利用Python的一个特殊的__slots__来实现。


{% highlight python %}
class Person(object):

    __slots__ = ('name', 'gender')

    def __init__(self, name, gender):
        self.name = name
        self.gender = gender

class Student(Person):

    __slots__ = ('name', 'gender', 'score')

    def __init__(self, name, gender, score):
        super(Student, self).__init__(name, gender)
        self.score = score

s = Student('Bob', 'male', 59)
s.name = 'Tim'
s.score = 99
print s.score

{% endhighlight %}


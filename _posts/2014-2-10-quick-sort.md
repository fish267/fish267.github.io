---
author: fish
layout: post
title: 快速排序--python
categories:
- Python
tags: algorithm
---
快速排序是C.R.A.Hoare于1962年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。

该方法的基本思想是：

1．先从数列中取出一个数作为基准数。

2．分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边。

3．再对左右区间重复第二步，直到各区间只有一个数。

其中Partion算法是算法的核心，可以从左右两端同时开始，这样快一些，示例图如下：![](undefined)
	<!--more-->

![](http://interactivepython.org/courselib/static/pythonds/_images/partitionA.png)

然后将选中的主元素放到中间：

![](http://interactivepython.org/courselib/static/pythonds/_images/partitionB.png)

算法代码如下：
{% highlight python %}    
    def quickSort(alist):
       quickSortHelper(alist,0,len(alist)-1)
      
    def quickSortHelper(alist,first,last):
       if first<last:
      
           splitpoint = partition(alist,first,last)
      
           quickSortHelper(alist,first,splitpoint-1)
           quickSortHelper(alist,splitpoint+1,last)
      
      
    def partition(alist,first,last):
       pivotvalue = alist[first]
      
       leftmark = first+1
       rightmark = last
      
       done = False
       while not done:
      
           while leftmark <= rightmark and \
                   alist[leftmark] <= pivotvalue:
               leftmark = leftmark + 1
      
           while alist[rightmark] >= pivotvalue and \
                   rightmark >= leftmark:
               rightmark = rightmark -1
      
           if rightmark < leftmark:
               done = True
           else:
               temp = alist[leftmark]
               alist[leftmark] = alist[rightmark]
               alist[rightmark] = temp
      
       temp = alist[first]
       alist[first] = alist[rightmark]
       alist[rightmark] = temp
      
      
       return rightmark
      
    alist = [54,26,93,17,77,31,44,55,20]
    quickSort(alist)
    print(alist)
{% endhighlight %}

---
author: fish
layout: post
title: 各种排序的Python实现
categories: Python
tags: algorithm
---
在博客园上看到一个Python的排序文章， [文章地址](http://www.cnblogs.com/wiki-royzhang/p/3614694.html）， 作者讲解的非常详细， 想学习的可以去看他的。 我此处留个笔记， 部分算法是按照自己的理解写的。
<!--more-->

包括冒泡， 插入， 选择， 归并， 快排， 堆排， 二叉树排， 桶排。 代码如下:<br>  
{% highlight python %}
#coding: utf-8
import random
import time

l = []
LEN = 50

for i in range(10):
    l.append(random.randint(1, 300))

global length
length = len(l)
print '*' * 100
print l
print '*' * 100

def check_len(l):
    if length == 0 or length == 1:
        return l
####################################
def bubbleSort(l):
    check_len(l)
    for i in xrange(len):
        for j in xrange(len - i - 1):
            if l[j] > l[j + 1]:
                tmp = l[j]
                l[j] = l[j + 1]
                l[j + 1] = tmp
    return l

####################################
def insertSort(l):
    check_len(l)
    for i in range(1, len):
        while i > 0 and l[i] < l[i -1]:
            tmp = l[i]
            l[i] = l[i - 1]
            l[i - 1] = tmp
            i -= 1
    return l
####################################
def selectSort(l):
    check_len(l)
    for i in xrange(len - 1):
        for j in range(i, len):
            if l[i] > l[j]:
                tmp = l[i]
                l[i] = l[j]
                l[j] = tmp
    return l
####################################
def mergeSort(L,start,end):
    check_len(l)
    def merge(L,s,m,e):
        left = L[s:m+1]
        right = L[m+1:e+1]
        while s<e:
            while len(left)>0 and len(right)>0:
                l[s] = left.pop(0) if left[0] < right[0] else right.pop(0)
                s+=1
            while len(left)>0:
                L[s] = left.pop(0)
                s+=1
            while len(right)>0:
                L[s] = right.pop(0)
                s+=1
            pass

    if start<end:
        mid = int((start+end)/2)
        mergeSort(L,start,mid)
        mergeSort(L,mid+1,end)
        merge(L,start,mid,end)
    return L
####################################

def quickSort(l):
    check_len(l)
    if len(l) <= 1:
        return l
    left = [i for i in l[1:] if i < l[0]]
    right = [i for i in l[1:] if i >= l[0]]
    return quickSort(left) + [l[0],] + quickSort(right)
####################################

def MinHeapFixdown(l, i, n):
    temp = l[i]
    j = 2 * i + 1
    while j < n:
        if j + 1 < n and l[j + 1] < l[j]:
            j += 1
        if l[j] >= temp:
            break
        l[i], i = l[j], j
        j = 2 * i + 1
    l[i] = temp
def MakeMiniHeap(l, n):
    i = n / 2 -1
    while i >= 0:
        MinHeapFixdown(l, i, n)
        i -= 1
    return l
def heapSort(l):
    if len(l) <= 1:
        return l
    t = length - 1
    while t > 0:
        l[t], l[0] = l[0], l[t]
        MinHeapFixdown(l, 0, t)
        t -= 1
    print  l


####################################
#¹¹Ôìstruct
class Node:
    def __init__(self, value = None, left = None, right = None):
        self.__value = value
        self.__left = left
        self.__right = right
    @property
    def value(self):
        return self.__value
    @property
    def left(self):
        return self.__left
    @property
    def right(self):
        return self.__right
def binaryTreeSort(l):
    check_len(l)
    class BinaryTree:
        def __init__(self, root = None):
            self.__root = root
            self.__ret = []
        @property
        def result(self):
            return self.__ret
        def add(self, parent, node):
            if parent.value < node.value:
                if not parent.left:
                    parent.left = node
                else:
                    self.add(parent.left, node)
            else:
                if not parent.right:
                    parent.right = node
                else:
                    self.add(parent.right, node)
        def Add(self, node):
            if not self.__root:
                self.__root = node
            else:
                self.add(self.__root, node)
        def show(self, node):
            if not node:
                return
            if node.right:
                self.show(node.right)
            self.__ret.append(node.value)
            if node.left:
                self.show(node.left)
        def Show(self):
            self.show(self.__root)
    b = BinaryTree()
    for i in xrange(length):
        b.Add(Node(l[i]))
    b.Show()
    return b.result

####################################
def countSort(l):
    check_len(l)
    max_num = max(l)
    storage = [0] * (max_num + 1)
    for i in xrange(length):
        storage[l[i]] += 1
    res = []
    for k in range(max_num + 1):
        if storage[k] != 0:
            for j in range(storage[k]):
                res.append(k)
    return  res

####################################
#print heapSort(MakeMiniHeap(l, length))
#print binaryTreeSort(l)
#print countSort(l)
{% endhighlight %}

#END

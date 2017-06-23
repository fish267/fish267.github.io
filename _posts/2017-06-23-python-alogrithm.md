---
author: Fish
layout: post
title: 数据结构--Python
categories: python
tags: algorighm
---

## Python 数据结构笔记

## 1. STACK

### 1.1 栈的定义

栈: [官方定义](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88), 一种特殊的串行数据结构, 只允许在一端进行加入数据与取出数据操作.

### 1.2 栈的例子
两个例子: 

- 桌面上的一摞书

![](https://zos.alipayobjects.com/rmsportal/zlZfGfcqzaYwkNNDATrj.png)

- 浏览器后退按钮

浏览器有个后退按钮, 每次点击, 跳转到最靠近当前页的 URL. (实际上, 进行的是 URLS 出栈操作).

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/d206f24af2a82bfa774e8918d854d653/image.png)

<!--more-->

### 1.3 栈的实现

栈遵循的是规则被称为 <code>LIFO, last-in first-out</code>, 具体的操作比较简单, 使用 Python List 强大的功能实现如下:

```python
class Stack:
    def __init__(self):
        self.items = []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        self.items.pop()

    def get_peek(self):
        return self.items[-1]

    def get_size(self):
        return len(self.items)

    def is_empty(self):
        return 0 == self.get_size()

    def display(self):
        print(self.items)
```

### 1.4 栈的应用

#### 1.4.1 校验四则运算表达式

计算机进行四则运算, 会先验证式子的格式正不正确, 下面对括弧判断一下:

```shell
{[()]}() --> True
{{[]}}[](} --> False

```

处理思路非常清晰, 存入左括号, 如果新增的右括号刚好凑对, 就消掉, 和玩**连连看**一个原理.

```python

def is_right_bracelet_format(s):
    stack = Stack()

    for element in s:
        if element in '{[(':
            stack.push(element)
        else:
            if stack.is_empty() or not bracelet_type_match(stack.get_peek(), element):
                return False
            stack.pop()
    return stack.is_empty()


def bracelet_type_match(left_type, right_type):
    left_type_list = ['(', '[', '{']
    right_type_list = [')', ']', '}']
    return left_type_list.index(left_type) == right_type_list.index(right_type)

```



#### 1.4.2 进制转换

# 2. Queue

类比与栈, 队列遵循的是规则被称为 <code>FIFO, first-in first-out</code>. 有个通俗的说法描述如下, 啊哈哈哈.

```shell
吃了吐是栈, 吃了拉是队列
```
## 2.1 队列的实现

队列分为分为普通队列和双端队列, 普通队列包括 **队尾入队** 和 **队头出队**, 双端队列两头都能进行 **入队/出队**.

普通队列实现, 按照从左到右的顺序, 左边是队尾.

```python
class Queue:
    def __init__(self):
        self.items = []

    def enqueue(self, item):
        self.items.insert(0, item)

    def dequeue(self):
        self.items.pop()

    def get_size(self):
        return len(self.items)

    def is_empty(self):
        return self.get_size() == 0

    def display(self):
        print(self.items)
```

双端队列, 多了个方法:

```python

class Deque:
    '''
        队列从左到右, 右边是队头
    '''

    def __init__(self):
        self.items = []

    def add_rear(self, item):
        self.items.insert(0, item)

    def remove_rear(self):
        self.items.pop(0)

    def add_front(self, item):
        self.items.append(item)

    def remove_front(self):
        self.items.pop()

    def get_size(self):
        return len(self.items)

    def is_empty(self):
        return self.get_size() == 0

    def display(self):
        print(self.items)
```

# 3. TREE

二叉树: Binary tree, 是每个节点最多只有两个分支的树结构.

## 3.1 二叉树实现

```python

class BinaryTree:
    def __init__(self, root_object):
        self.key = root_object
        self.left_child = None
        self.right_child = None

    def insert_left(self, node):
        if self.left_child is None:
            self.left_child = BinaryTree(node)
        else:
            tmp_node = BinaryTree(node)
            tmp_node.left_child = self.left_child
            self.left_child = tmp_node

    def insert_right(self, node):
        tmp_node = BinaryTree(node)
        if self.right_child is None:
            self.right_child = tmp_node
        else:
            tmp_node.right_child = self.right_child
            self.right_child = tmp_node

    def get_left_child(self):
        return self.left_child

    def get_right_child(self):
        return self.right_child

    def get_root(self):
        return self.key
```

### 二叉树的遍历

```python
ft_child():
        pre_order_display(tree.get_left_child())

    if tree.get_right_child():
        pre_order_display(tree.get_right_child())


# 中序遍历
def in_order_display(tree):
    if not isinstance(tree, BinaryTree):
        return False

    if tree.get_left_child():
        in_order_display(tree.get_left_child())

    print(tree.key)

    if tree.get_right_child():
        in_order_display(tree.get_right_child())


# 后序遍历
def post_order_display(tree):
    if not isinstance(tree, BinaryTree):
        return False

    if tree.get_left_child():
        post_order_display(tree.get_left_child())

    if tree.get_right_child():
        post_order_display(tree.get_right_child())

    print(tree.key)

# 翻转

def invert_binary_tree(tree):
    tree.left_child, tree.right_child = tree.right_child, tree.left_child
    if tree.get_left_child() != None:
        invert_binary_tree(tree.get_left_child())
    if tree.get_right_child() != None:
        invert_binary_tree(tree.get_right_child())
```

@TODO


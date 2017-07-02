---
author: Fish
layout: post
title: 一张打折卡引发的精度思考
categories: python 
tags: work
---


## 1. 背景

在验证网商营销平台时, 发现一个奇怪的打折卡

![Screen_Shot_2016-08-10_at_21.46.55](https://gw.alipayobjects.com/zos/rmsportal/tOpYUuQAQowIzwAoCeeq.png)

经排查, 服务端返回的是打折卡额度分别是 0.39, 0.49, 前端数值乘以 10 进行展示, <code>0.39 * 10 = 3.9000000000000004</code>, 在浏览器演示如下:

![Screen_Shot_2016-08-10_at_21.51.00](https://gw.alipayobjects.com/zos/rmsportal/NwvSVWCOMyGammHpkFXe.png)

很明显是 Javascript 浮点运算精度问题, 那么为什么两个运算结果一个正常, 一个这么不正经呢?

<!--more-->

## 2. Python 分析

Javascript 和 Python 都是弱类型语言, 以<code>0.1 + 0.2</code>为例, 从更为熟悉的 Python 分析一下. 

在 CPython 解释器下, 得到的结果如下:

![Screen_Shot_2016-08-10_at_22.14.15](https://gw.alipayobjects.com/zos/rmsportal/NENdkgykvdDkkPvPMWem.png)

我们知道计算机都是二进制运算, 通过[进制转换工具](http://tool.oschina.net/hexconvert/), 或者文章末尾的 Python 计算函数<code>get_float_bin()</code>, 可以看到运算过程: 

```python
0.1 的二进制: 0.0001100110011001100110011001100110011001100110011001101
0.2 的二进制: 0.0011001100110011001100110011001100110011001100110011010
二进制相加:	 0.0100110011001100110011001100110011001100110011001100111

转换成浮点:   0.3000000000000000444089209850062616169452667236328125

截断后:		 0.30000000000000004
```
浮点数有指数和尾数组成, 在转换过程中, 无限长度的二进制数过程不可逆, 造成了精度丢失.

至于上面的<code>0.49 * 10 = 4.9</code>, 可以推测, 相乘之后的二进制转成10进制, 恰好9后面有至少17个零, 这样 CPython 解释器就以4.9展示了.

#### Python 眼中的 0.1
通过 <b>Decimal</b> 模块, 我们看看 Python 是怎么看待 0.1 的, 比我们要费脑多了.
```python
>>> from decimal import Decimal
>>> Decimal(0.1)
Decimal('0.1000000000000000055511151231257827021181583404541015625')
```
这个非常有意思, 也正好解答了许久以来的困惑, 为什么 <code>Python(round)、Javascript(toFixed) </code> 等语言的四舍五入不灵!!!

- Python3


```python
>>> round(3.155, 2)
3.15
>>> round(2.615, 2)
2.62
```

- Node

```python
> console.log(2.675.toFixed(2))
2.67
undefined
> 2.615.toFixed(2)
'2.62'
```

Python 和 Js 准守的是标准的四舍五入, 不存在(叫奇进偶舍), 至于为什么 3.155 没有五入, 我们看看 Python 是怎么看 3.155 的:

```python
>>> from decimal import Decimal
>>> Decimal(3.155)
Decimal('3.154999999999999804600747665972448885440826416015625')
```

很显然, 他四舍五入取两位时, 肯定就是 <b>3.15</b>了!!

## 3. 解决方案

BB 了这么多, 吐槽一下解决方案: 

#### 方案一: 统统转换成整数
1. 二进制表示整数是没有精度丢失的, 浮点数的运算, 统一乘以 10 的 N 次方, 变成整数来运算

#### 方案二: BigDecimal
2. 《Effective Java》 说了, 商业的计算, 要用 Java.math.BigDecimal, 前端就负责的渲染展示, 数值计算统统扔给服务端来保证吧!
	
既然用到了 BigDecimal, 顺便提一下, 我们之前踩过一个坑, Double 类型直接转成 BigDecimal 类型去用:

```java 
public class TestLab {
    public static void main(String[] args){
        Double num = 996.007;
        System.out.println(num);
        BigDecimal bgNum = new BigDecimal(num);
        System.out.println(bgNum);
    }
}
```

输出如下: 

	996.007
	996.0069999999999481588019989430904388427734375 


WTF! 正确的姿势应该是

```java
BigDecimal bgNum = new BigDecimal(num.toString());
```

<b>牢记 BigDecimal 只能精确的将字符串转成浮点!</b>


## 4. 其他

### 1. Python 计算 float 的二进制
 
```python
from decimal import Decimal

def get_float_bin(x, n=50):
    a = Decimal(x) * 2
    r = []
    for i in range(0, n):
        if a >= 1:
            r.append("1")
            a -= 1
        else:
            r.append("0")
        a = a * 2
    return "0." + "".join(r)

// 结果
>>> get_float_bin(0.1)
'0.00011001100110011001100110011001100110011001100110'
```

### 参考文档

1. [stackoverflow 问题](http://stackoverflow.com/questions/8073912/why-do-we-need-to-convert-the-double-into-a-string-before-we-can-convert-it-int[])
2. [Javascript中浮点数的计算精度问题](http://blog.gejiawen.com/2015/08/11/javascript-float-count-precision/)
3. [浮点数的精度](http://blog.chinaunix.net/xmlrpc.php?r=blog/article&uid=26731984&id=3110102)

<hr>
##### 希望 Wings 战队 Ti6 夺冠.

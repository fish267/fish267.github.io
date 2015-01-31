---
author: Fish
layout: post
title: OpenMP学习笔记<一>
categories: C++
tags: openmp
---
##并行程序设计思路
并行程序设计和串行的稍微不同, 大概经历下面四个阶段:
    <b>划分</b>: 将计算分解成尽可能多的小任务, 可以按处理对象数据进行域分解, 也可以按计算任务进行功能分解. 这两种分解要做到数据集和计算集不相交.
    <b>通信</b>: 划分产生的任务, 一般不能完全独立执行, 需要进行数据通信
    <b>组合</b>: 根据任务的局部性, 将小任务组合成更大的任务
    <b>映射</b>: 将组合后的多个任务分配到多个处理器上, 并期望获得算法的最优性能减少算法的执行时间

#OpenMp 基本概念
OpenMP 属于共享内存编程模型的技术, 通过在源代码中添加指导注释, 成为编译指导: <code> #pragma omp parallel </code>


OpenMP 支持的语言包括 C/C++ 和 Fortran, 支持开源编译器 gcc/g++, 下面首先了解OpenMP 的执行模式和三大要素.
<!--more-->
##执行模式
OpenMP 采用 fork-join 的模式, 其中 fork 创建新线程或者唤醒已有的线程, jion 即多线程的会合. 主线程在运行过程中, 需要并行计算的时候, 派生出线程来执行并行任务. 在并行代码执行结束后, 派生线程退出或阻塞, 控制流程回到主线程上. 
##编程要素
###1. 编程指导
在 C/C++ 中, OpenMP 的编译指导指令是以 <code> #pragma omp </code> 开头的, 形式如下:
{% highlight cpp %}
#pragma omp 指令 [子句 [子句]....]
{% endhighlight %}
指导指令和子句按照功能大体分为四类:
    并行域控制类
    任务分担类
    同步控制类
    数据环境类


    
***下面是 OpenMP 规范的指令:***


    parallel* :    用在一个结构块之前，表示这段代码将被多个线程并行执行
    for* :    用于for循环语句之前，表示将循环计算任务分配到多个线程中并行执行，以实现任务分担，必须由编程人员自己保证每次循环之间无数据相关性
    parallel for* :    parallel和for指令的结合，也是用在for循环语句之前，表示for循环体的代码将被多个线程并行执行，它同时具有并行域的产生和任务分担两个功能
    sections* :    用在可被并行执行的代码段之前，用于实现多个结构块语句的任务分担，可并行执行的代码段各自用section指令标出
    parallel sections* :    parallel和sections两个语句的结合，类似于parallel for
    single* :    用在并行域内，表示一段只被单个线程执行的代码
    critical* :    用在一段代码临界区之前，保证每次只有一个OpenMP线程进入
    flush* :    保证各个OpenMP线程的数据影像的一致性
    barrier* :    用于并行域内代码的线程同步，线程执行到barrier时要停下等待，直到所有线程都执行到barrier时才能继续往下执行
    atomic* :    用于指定一个数据操作需要原子性地完成
    master* :    用于指定一段代码由主线程执行
    threadprivate* :    用于指定一个或多个变量是线程专用。

***相应的 OpenMP 子句:***


    private* :    指定一个或多个变量在每个线程中都有它自己的私有副本
    firstprivate* :    指定一个或多个变量在每个线程都有它自己的私有副本，并且私有变量要在进入并行域或认为分担域时，继承主线程中的同名变量的值作为初值
    lastprivate* :    是用来指定将线程中的一个或多个私有变量的值在并行处理结束后复制到主线程中的同名变量中，负责拷贝的线程是for或sections任务分担中的最后一个线程
    reduction* :    用来指定一个或多个变量是私有的，并且在并行处理结束后这些变量要执行指定的归约运算，并将结果返回给主线程同名变量
    nowait* :    指出并发线程可以忽略其他制导指令暗含的路障同步
    num_threads* :    指定并行域内的线程的数目
    schedule* :    指定for任务分担中的任务分配调度类型
    shared* :    指定一个或多个变量为多个线程间的共享变量
    copyprivate* :    配合single指令，将指定线程的专有变量广播到并行域内其他线程的同名变量中
    copyin* :    用来指定一个threadprivate类型的变量需要用主线程同名变量进行初始化
    default* :    用来指定并行域内的变量的使用方式，缺省是shared。
## API 函数

    
    omp_in_parallel:    判断当前是否在并行域中
    omp_get_thread_num:    返回线程号
    omp_set_num_threads:    设置后续并行域中的线程个数
    omp_get_num_threads:    返回当前并行区域中的线程数
    omp_get_max_threads:    获取并行域可用的最大线程数目
    omp_get_num_procs:    返回系统中处理器个数
    omp_get_dynamic:    判断是否支持动态改变线程数目
    omp_set_dynamic:    启用或关闭线程数目的动态改变
    omp_get_nested:    判断系统是否支持并行嵌套
    omp_set_nested:    启用或关闭并行嵌套
    omp_init(_nest)_lock:    初始化一个(嵌套)锁
    omp_destroy(_nest)_lock:    销毁一个(嵌套)锁
    omp_set(_nest)_lock:    (嵌套)加锁操作
    omp_unset(_nest)_lock:    (嵌套)解锁操作
    omp_test(_nest)_lock:    非阻塞的(嵌套)加锁
    omp_get_wtime:    获取wall time时间
    omp_set_wtime:    设置wall time时间。
##环境变量
<code>OMP_SCHEDULE</code>: 只能用到 for, parallel for中。它的值就是处理器中循环的次数


<code>
OMP_NUM_THREADS</code>: 定义执行中最大的线程数


<code>
OMP_DYNAMIC</code>: 通过设定变量值TRUE或FALSE, 来确定是否动态设定并行域执行的线程数


<code>
OMP_NESTED</code>: 确定是否可以并行嵌套

#END

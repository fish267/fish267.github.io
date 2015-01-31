---
layout: post
author: Fish 
title: Linux基本查询信息
categories: Shell 
tags: linux 
---
阅读别人的Python代码时， 留意了一下他对 cpu 使用率， 内存占用的计算流程，就记录在了笔记上， 现在分享一下。


无非是 Linux 自带的命令显示， 文章末给出了经常应用的一个 cheat~
#检测内存信息
在终端输入<code> free </code>, 出来一大堆:

	fish@Love67:~$ free
		     total       used       free     shared    buffers     cached
	Mem:       3857400    2868708     988692          0     288856    1322584
	-/+ buffers/cache:    1257268    2600132
	Swap:      3999740          0    3999740

<!--more-->
在 free 命令添加参数 -m ,是以 MB 单位现实内存的大小， 很明显， 我们需要的是第二行 Mem 的前两列数据。使用管道命令，
<code>grep -i mem</code>, <code>-i </code> 是忽略大小写。

	Mem:       3857400    2868708     988692          0     288856    1322584

然后，使用 awk 命令来摘选出前两个数据， 看看 gawk 的定义：

	awk: reads standard input and writes standard output

综上，具体命令如下:
{% highlight sh %}
free -m | grep -i mem | awk {'print $1'}
{% endhighlight %}
上面 $0 代表的是整个一行， $1 是第一个数据，依次往后~
#检测 CPU 信息
本来是想着写 <code> cat /proc/stat </code> 的方法来着， 原代码就是根据显示的数据进行采样， 然后计算 cpu 使用率。
刚刚觉得， 还是使用 top 命令吧， 这个耳熟能详的，平易近人。


只更新一次就退出的 top 指令：

	top -n 1

显示 cpu 的使用率如下：
{% highlight sh %}
top -n 1 | grep -i cpu | awk {'print $2'}
{% endhighlight %}
其实这么说来， 利用 top 命令显示内存也更容易了:
{% highlight sh %}
top -n 1 | grep Men | awk {'print $4'}
{% endhighlight %}

#下面是速查命令
##系统
{% highlight sh %}
# uname -a		# 查看内核/操作系统/CPU信息
# cat /etc/issue	# 查看操作系统版本
# hostname		# 查看计算机名
# cat /proc/cpuinfo	# 查看CPU信息
# lspci -tv              # 列出所有PCI设备
# lsusb -tv              # 列出所有USB设备
# lsmod                  # 列出加载的内核模块
# env                    # 查看环境变量
{% endhighlight %}
##资源
{% highlight sh %}
# free -m                # 查看内存使用量和交换区使用量
# df -h                  # 查看各分区使用情况
# du -sh <目录名>        # 查看指定目录的大小
# grep MemTotal /proc/meminfo   # 查看内存总量
# grep MemFree /proc/meminfo    # 查看空闲内存量
# uptime                 # 查看系统运行时间、用户数、负载
# cat /proc/loadavg      # 查看系统负载
{% endhighlight %}
##磁盘和分区
{% highlight sh %}
# mount | column -t      # 查看挂接的分区状态
# fdisk -l               # 查看所有分区
# swapon -s              # 查看所有交换分区
# hdparm -i /dev/hda     # 查看磁盘参数(仅适用于IDE设备)
# dmesg | grep IDE       # 查看启动时IDE设备检测状况
{% endhighlight %}
##网络
{% highlight sh %}
# ifconfig               # 查看所有网络接口的属性
# iptables -L            # 查看防火墙设置
# route -n               # 查看路由表
# netstat -lntp          # 查看所有监听端口
# netstat -antp          # 查看所有已经建立的连接
# netstat -s             # 查看网络统计信息
{% endhighlight %}
##进程和服务
{% highlight sh %}
# ps -ef 		 # 查看所有进程
# top			 # 动态显示进程

# chkconfig --list	 # 列出所有系统服务
# chkconfig --list | grep on	# 启动的服务
{% endhighlight %}
##用户
{% highlight sh %}
# w                      # 查看活动用户
# id <用户名>            # 查看指定用户信息
# last                   # 查看用户登录日志
# cut -d: -f1 /etc/passwd   # 查看系统所有用户
# cut -d: -f1 /etc/group    # 查看系统所有组
# crontab -l             # 查看当前用户的计划任务
{% endhighlight %}
#END

---
layout: post
author: fish
title: 系统化详解Linux命令(译)
categories: Shell
tags: linux
---
Linux命令本身就是linux使用者学习linux命令的一个快速参考, 命令可以分为15个种类, 这样按需所求, 更加助于理解. PDF版本的 linux 命令也可以搞到. 你如果对本教程有任何评价或者指正, 可以联系作者 [Bobbin Zachariah](https://plus.google.com/115113980420145314347/posts)

你可以点击[下载](http://www.linoxide.com/doc/linux_command_shelf_pdf_ver1_1.pdf)最新的PDF版linux命令. 目前最新版是1.1, 本教程适用于linux新手和老鸟, 尽最大努力详解常用的linux命令.

你可以跳过某章节, 直接根据导航去查询. 如果你对某些命令感到困惑, 请去我的简历页面告知.
<!--more-->
<hr>

1. [系统](#t1)
2. [硬件](#t2)
3. [数据](#t3)
4. [用户](#t4)
5. [文件命令](#t5)
6. [进程相关](#t6)
7. [文件权限操作](#t7)
8. [网络](#t8)
9. [压缩/解压](#t9)
10. [安装包](#t10)
11. [查询](#t11)
12. [登陆](#t12)
13. [文件传输](#t13)
14. [硬盘使用](#t14)
15. [文件夹跳转](#t15)

<hr>
#1. 系统
    
    $ uname -a                              =>系统信息
    $ uname -r                              =>显示内核信息
    $ cat /etc/redhat_release               =>安装的是哪个版本
    $ uptime                                =>显示系统运行了多久
    $ hostname                              =>系统host名称
    $ hostname -i                           =>host IP
    $ last reboot                           =>系统重启历史
    $ date                                  =>时间信息
    $ cal                                   =>显示月份信息
    $ w                                     =>在线用户
    $ whoami                                =>你以什么身份登陆
    $ finger user                           =>显示用户信息

<hr>
#2. 硬件

    $ dmesg                                 =>检测硬件和启动信息
    $ cat /proc/cpuinfo                     =>CPU 模块
    $ cat /proc/meminfo                     =>内存信息
    $ cat /proc/interrupts                  =>每个I/O设备CPU的中断数量
    $ lshw                                  =>硬件配置信息
    $ lsblk                                 =>linux 设备块信息
    $ free -m                               =>内存使用量(-m 以MB为单位)
    $ lspci -tv                             =>PCI 设备信息
    $ lsusb -tv                             =>USB 设备
    $ lshal                                 =>显示设备和他们的熟悉
    $ dmidecode                             =>从BIOS显示硬件信息
    $ hdparm -i /dev/sda                    =>显示硬盘 sda 的信息
    $ hdparm -tT /dev/sda                   =>对硬盘 sda 做读取测试
    $ badblocks -s /dev/sda                 =>测试硬盘 sda 的坏道

<hr>
#3. 统计信息

    $ top                                   =>动态显示 cpu 进程
    $ mpstat 1                              =>显示进程相关数据
    $ vmstat 2                              =>显示虚拟内存使用数据
    $ iostat 2                              =>I/O 数据
    $ tail -n 500 .vimrc                    =>最后500行信息
    $ tcpdump -i eth0                       =>抓eth0所有的数据流
    $ tcpdump -i eth0 'port 80'             =>监听80口的流量
    $ lsof                                  =>列出所有活动进程文件
    $ lsof -u testuer                       =>列出被用户 testuser 打开的文件
    $ watch df -h                           =>实时监控变化的数据

<hr>
#4. 用户 
	$ id                                    =>显示当前用户的登陆信息
	$ last                                  =>最近的系统登陆
	$ who                                   =>登陆用户名
	$ groupadd admin                        =>添加用户组 admin
	$ useradd -c 'fish' -g admin -m sam     =>创建用户'sam'并添加到amdin用户组
	$ userdel sam                           =>删除用户sam
	$ adduser sam                           =>添加用户sam
	$ usermood                              =>修改用户信息
	
<hr>
#5. 文件命令

	$ ls -al                                =>显示目录下所有文件的信息
	$ pwd                                   =>当前所在路径
	$ mkdir directory-name                  =>创建文件夹
	$ rm file-name                          =>删除文件
	$ rm -r directory-name                  =>递归删除文件夹
	$ rm -f file-name                       =>强行删除文件
	$ rm -rf directory-name                 =>强行递归删除文件夹
	$ cp file1 file2                        =>复制文件file1到file2
	$ cp -r dir1 dir2                       =>复制dir1到dir2， 如果不存在就创建dir2
	$ mv file1 file2                        =>移动
	$ ln -s /path/to/file-name link-name    =>创建软连接
	$ touch file                            =>创建文件
	$ cat > file                            =>标准输出流到file
	$ more file                             =>显示文件内容
	$ head file                             =>默认打印前十行信息
	$ tail file                             =>显示最后10行信息
	$ tail -f file                          =>动态显示最后10行
	$ gpg -c file                           =>授权file
	$ gpg file.gpg                          =>取消授权
	
<hr>
#6. 进程相关

    ps                                      =>显示活动进程
    ps aux | grep 'telnet'                  =>显示所有与'telnet'有关的进程
    pmap id                                 =>进程内存信息
    top                                     =>实时显示进程
    kill pid                                =>杀死进程
    killall proc                            =>杀死所有名为 proc 的进程
    pkill   processname                     =>根据进程名杀死进程
    bg                                      =>将程序放到后台
    fg                                      =>将最近的一个后台程序提到前台
    fg n                                    =>n个任务提到前台

<hr>
#7. 文件权限操作

    chmod octal file-name                   =>设置文件权限为otcal
    chmod 777 /data/test.c                  =>文件权限rwxrwxrwx
    chmod 755 /data/test.c                  =>文件权限rwxr-xr-x
    chown owener-user file                  =>改变拥有者
    chown owener-user:owner-group file-name =>设置文件用户组和用户名
    chown owener-user:owener-group dir      =>设置文件夹的用户组和用户名

<hr>
#8. 网络

    ifconfig -a                             =>显示网络接口和ip信息
    ifconfig eth0                           =>显示网卡eth0的信息
    ip addr show                            =>显示所有网络接口和ip地址
    ip address add 192.168.0.1 dev eth0     =>设置ip
    ethtool eth0                            =>linux自带工具显示网卡信息
    mii-tool eth0                           =>同上
    ping host                               =>向host发送测试包
    whois domain                            =>得到域名信息
    dig domain                              =>获得DNS信息
    dig -x host                             =>反向查询host
    host google.com                         =>查询DNS, ip 
    hostname -i                             =>查询本地Ip
    wget file                               =>下载文件
    netstat  -tup1                          =>监听活动端口(tcp, upd, pid)

<hr>
#9. 解压/压缩

    tar cf home.tar home                    =>压缩 home 文件夹为 home.tar
    tar xf file.tar                         =>解压文件file.tar
    tar czf file.tar.gz files               =>压缩成 gz 版
    gzip file                               =>将文件压缩成file.gz

<hr>
#10. 安装包

    rpm -i pkgname.rpm                      =>安装命令
    rpm -e pkgname                          =>移除程序
    源码安装
    ./configure 
    make
    make install

<hr>
#11. 查询

    grep pattern files                      =>在文件中查询 pattern
    grep -r pattern dir                     =>递归在文件夹中查找 pattern
    locate file                             =>查找所有文件
    find /home/tom -name 'index*'           =>在指定文件夹下查找 index 开头的文件
    find /home -size +10000k                =>查找指定文件夹下大于10000k的文件

<hr>
#12. 登陆
    
    ssh user@host                           =>ssh 登陆命令
    ssh -p port user@host                   =>ssh登陆指定端口的
    telnet host                             =>使用telnet 端口登陆

<hr>
#13. 文件传输

    scp file.txt server2:/tmp               =>向server传送file.txt
    scp nixsavy@serveer2:/www/*.html ./     =>重server下载文件
    scp -r nixsavy@server2:/www/* ./        =>递归下载
    rsync -a /home/apps  /backup/           =>同步两个文件夹内容
    rsync -avz /home/apps user@server:/dir  =>同步服务器上的文件夹

<hr>
#14. 硬盘使用

    df -h                                   =>挂载硬盘的使用信息
    df -i                                   =>挂载硬盘的inode
    fdisk -l                                =>硬盘分区和类型
    du -ah                                  =>所有文件大小信息
    du -sh                                  =>当前目录总大小
    findmnt                                 =>显示文件系统
    mount device mount-point                =>挂载设备

<hr>
#15. 文件夹跳转

    cd ..                                   =>跳转到上一目录
    cd                                      =>去 $home 目录
    cd /test                                =>跳转到 /test


[原文地址](http://linoxide.com/guide/linux-command-shelf.html), 试着翻译着玩, 如果你不小心看到了, 又发现了错误, 谢谢留言指正!

#END

    

---
author: fish
layout: post
title: Shell 交互工具 expect
categories: Shell
tags: linux
---

执行该命令时，提示需要输入密码，谷歌了一下发现有expect这个工具，可以进行脚本自动交互，填入数据。直接贴个简单登录代码，后面部分可以忽略不看:
{% highlight python %}  
#!/usr/bin/expect  -f
spawn ssh root@$facecode.org
expect "password:"
send "xxoo\r"
interact
{% endhighlight %}
<!--more-->
## 1. expect 是什么
**expect是用来和shell进行自动交互的**。比如ssh server需要手动输入passwd，用expect就能自动登录了。
在server上输入```man expect```会出来最详细的expect英文介绍，或者看这里 [expect(1) - Linux man page][2]。由于内容实在太多，挑个最简单的来解释
## 2. expect 使用
###2.1 **安装expect**
{% highlight python %}    
yum install expect 
{% endhighlight %}
另外，执行
{% highlight python %}    
$ which expect
/usr/bin/expect
{% endhighlight %}
可以找到执行路径，类似python shell文件，需要在第一行添加 ```#!/usr/bin/expect  ```命令
###2.2  **编辑脚本**
{% highlight python %}    
#!/usr/bin/expect  
set url [lindex $argv 0]
set timeout 30
spawn ssh log@$url
expect "password:"
sleep 1
send "hahahhaha\r"
interact
{% endhighlight %}
上面这个脚本是用来登录server的，有了这个后，执行下面的命令
{% highlight python %}    
ssh log@**.xx.net 
ssh admin@**.xx.net
{% endhighlight %}
就不需要输入蛋疼的密码了（尤其是输错几次）

* 第一行不解释
* 第二行 [lindex $argv 0] 用于获取 终端的输入
* 第三行设置超时时间，比如登录、执行等命令都有延迟
* spawn 是expect下的命令，它主要的功能是给ssh运行进程加个壳，用来传递交互指令。
* ```expect "password:" ```返回命令中包含'password'字符串，如果有就返回，等待时间为刚才设置的超时时间
* ```send "hahahhaha\r"``` 这个就是替代人来进行的输入
* ```intaract``` 执行完成后保持交互状态，把控制权交给控制台

###2.3  **修改脚本权限，放到可执行目录下**


感谢涵光的问题，起码鼓捣出这个脚本，以后再也不用输入xixihaha了！！！
  [1]: http://atit.alipay.net/index.php?r=blog/detail&qid=2280
  [2]: http://linux.die.net/man/1/expect

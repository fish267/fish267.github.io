---
author: Fish
layout: post
title: Raspberry Usage
categories: diy 
tags: linux
---

久闻树莓派之大名，600大洋购于闲鱼，2020年8月4日有幸相会，开箱一刻，很是惊艳，原来就是身份证大小的玩意儿，捆绑销售一堆不明觉厉小器件。

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*sUjsSK3MkCIAAAAAAAAAAABkARQnAQ)

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*meTkS7z2eHAAAAAAAAAAAABkARQnAQ)

<!--more-->

# 折腾之旅

是日也，天朗气清，惠风和畅，开始长达两周的 <code>Raspberry Pi 4B+</code> 折腾之旅，谨以此文记录下!


## 1. 系统安装与登录

**"虽趣舍万殊，静躁不同，当其欣于所遇，暂得于己，快然自足，不知老之将至" 	---《兰亭集序》**

<hr></hr>

1. 系统初始化简单，读卡器插到电脑上，直接用  Raspberry Pi Imager 软件进行安装 [下载地址](https://www.raspberrypi.org/downloads/)。也可以自己下载镜像后，img镜像烧录进去，用的是 etcher 软件，这种方式比较麻烦，适合安装非官方镜像，不建议使用;
2. 取出TF卡，插到树莓派上，通过 micro DP 连上显示器，原则上就可以开机了。我没有这么做，一是没多余的显示器，二是键鼠都是蓝牙的，初次进入系统毫无办法，用下面三种方式都可以：
	
	- 打开tf卡，新建ssh空白文件，通过网线连到路由器，ssh连接树莓派
	- 打开tf卡，新建ssh空白文件，通过网线连接到电脑，电脑共享网络后，ssh连接树莓派
	- 打开tf卡，新建ssh空白文件与 wpa_supplicant.conf 文件，自动连上wifi

3. 推荐用第三种方案，不局限于网线的限制，玩硬件时摆出各种姿势，wpa_supplicant.conf 文件内容如下，ssid与psk分别是wifi名称与密码

```java
country=CN

ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

update_config=1

network={
	ssid="python"
	psk="251125251"
	priority=2
}
```

在路由器上看看树莓派 IP，然后ssh就可以访问了，默认账密是 pi/raspberry

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*jvFuTLI08kgAAAAAAAAAAABkARQnAQ)

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*LPUoSrl15J0AAAAAAAAAAABkARQnAQ)

注意登录信息上有 armv7l，表示该板子是 arm 架构，后面玩 docker 时会有不少坑。

家里有公网IP的话，路由器上做一下端口转发，就可以随时随地操控树莓派了，和买了个独立VPS一样，我将ssh的22端口，映射到了5267端口上，下图是用手机端 termius APP访问的截图

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*NYZNTZEJ6PQAAAAAAAAAAABkARQnAQ)

## 2. 初始化 Linux 环境

**Across the Great Wall we can reach every corner in the world. ---国内第一封邮件**

<hr></hr>

- 拿到新VPS后，先不要安装，第一件事就是换源，将清华的源覆盖掉官方源

```shell
echo 'deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi' > /etc/apt/sources.list

echo 'deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui deb-src http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui' > /etc/apt/sources.list.d/raspi.list
```

- 然后进行 <code>zsh/git/autojump/vim</code>等常用工具安装，记得安装下 xrdp，然后映射下 3389 端口号到路由器

```shell
sudo apt install xrdp
```

使用 Remote DeskTop 等远程桌面软件，就可以访问到树莓派的图形化系统了，看看就行，没多大用，还占内存, 我唯一的用途是登录上去，打开家里路由器，添加端口号映射

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*1aO_TZpG4HsAAAAAAAAAAABkARQnAQ)



## 3. 初玩硬件

** 仰观宇宙之大，俯察品类之盛，所以游目骋怀，足以极视听之娱，信可乐也 **

<hr></hr>


板子上有个40Pin 的针脚，将他用延长线接出来，插到面包板上，玩一玩捆绑销售的硬件。

使用树莓派编程，需要了解 GPIO，非常非常简单，直接根据针脚图，找到编号，然后写代码控制编号对应的针脚，输出高电平或者低电平即可，比当年51单片机省事多了。

![](https://shumeipai.nxez.com/wp-content/uploads/2015/03/rpi-pins-40-0.png)

下图是连接面包板的, 比较凌乱

![](https://ftp.bmp.ovh/imgs/2020/08/64e544e1831ed13a.jpg)

看下面的代码，控制发光二极管忽闪，只需要将编号17的针脚与外设连一起，就完事了。代码如此简单，令人发指，完全不用关心驱动，铁憨憨一样。

```python
import RPi.GPIO as GPIO
import time

pin = 17
GPIO.setmode(GPIO.BCM)
GPIO.setup(pin, GPIO.OUT)
GPIO.output(pin, GPIO.HIGH)

while True:
    GPIO.output(pin, GPIO.LOW)
    time.sleep(2)
    print('lighting ~')
    GPIO.output(pin, GPIO.HIGH)
    time.sleep(2)
```

高级一点，我连上两个硬件, 一个按钮与激光头，实现了激光笔功能，按下开关就亮，代码也是清晰明了, 毫无技术含量.

可见树莓派是个纯软件玩具，再加上简单的 python，门槛极低，自带图形化界面，还有拖拽式编程，特别适合5岁及以上的孩子上手。

```python
import RPi.GPIO as GPIO
import time

pin = 17
GPIO.setmode(GPIO.BCM)
GPIO.setup(pin, GPIO.OUT)
GPIO.output(pin, GPIO.HIGH)

button_pin = 18
GPIO.setup(button_pin, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.output(pin, GPIO.HIGH)
while True:
    value = GPIO.input(button_pin)
    print(value)
    time.sleep(0.1)
    if(value == 0):
        print('lighting ~')
        GPIO.output(pin, GPIO.HIGH)
    else:
        GPIO.output(pin, GPIO.LOW)
```

首次点亮二极管，还是挺兴奋的，我认为这才是树莓派的初衷，如果有公益项目，普及计算机给广大的乡村，该玩具肯定能给孩子带来无限乐趣与灵感，未来的世界，编程、外语等技能会和说普通话一样简单普及。

后之视今，亦犹今之视昔~



## 4. 当做网盘

**道生一，一生二，二生三，三生万物--《道德经》
**
<hr></hr>

我心目中的网盘，就是免费、开源、多账户、多客户端，简单说就是和百度网盘VIP中P一样的效果，最终选型 seafile，果然不负所托，非常好用。

安装方式参考 [seafile安装文档](https://cloud.seafile.com/published/seafile-manual-cn/deploy/using_sqlite.md)，不要折腾 docker 了，没有适合 arm 的镜像。

 - 启动7.1.4版本时, 遇到了问题, 在论坛上解决的 [https://bbs.seafile.com/t/topic/12353](https://bbs.seafile.com/t/topic/12353)

- 一般树莓派做网盘，都会外接硬盘，数据存放到硬盘上，一是直接按照到移动硬盘

- 第二种方案是给 seafile-data 创建软连接，指向移动硬盘的目录，推荐第一种方案，简单好迁移，缺点是页面处理速度慢一点。

硬盘丰富的话，记得对 seahub-data 目录进行 rsync 备份，没盘的可以同步到网盘。


```shell
➜  seafile tree . -L 1
.
|-- ccnet
|-- conf
|-- logs
|-- pids
|-- seafile-data
|-- seafile-server-7.1.4
|-- seafile-server-latest -> seafile-server-7.1.4
|-- seahub-data
`-- seahub.db

```

启动 seafile.sh 与 seahub.sh 后，默认占用8000端口，在设置项中，勾选"允许用户注册"。 PC页面与APP截图如下，简单粗暴且强大

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*EKrWS5AG7AwAAAAAAAAAAABkARQnAQ)

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*w0k2TJd3zrcAAAAAAAAAAABkARQnAQ)

## 5. 当Web相册

**韶华易逝,时光荏苒,流年不在**

<hr></hr>

照片是岁月留痕，调研了几个web相册，最终选型了 piwigo，古老的PHP代码编写，支持丰富的插件与客户端（ios客户端没连上）

该应用折腾了好久 run 起来，docker 也不要折腾了，也是根本适配不了树莓派。由此可见，纯粹的折腾软件，还是x86的小型机，拥有无可比拟的优势。。

根据官方文档，[安装方式地址](https://cn.piwigo.org/get-piwigo)，需要提前安装好 nginx与php、mysql(树莓派只能安装 MariaDB)

**踩坑记录:**

该软件是折腾最耗时的, 我接入了移动硬盘后, 建立软连接也始终无法上传到硬盘上. 最终部署到了移动硬盘中, 但是上传文件时, 又遇到了报错, 原因是 fpm与nginx 使用的用户组是 www-data, 无法访问挂载的硬盘, 解决方式:

1. 修改 nginx 用户组, /etc/nginx/nginx.conf 配置文件, 修改 user www-data; 为 user pi;
2. 修改 fpm 用户组, /etc/php/7.3/fpm/pool.d/www.conf, 将 user与group 统一改成 pi, 需要改4处
2. 直接运行会报错，需要改下代码，[参考文档](https://dvpizone.wordpress.com/2014/03/11/how-to-install-piwigo-on-a-raspberry-pi/#:~:text=How%20to%20install%20Piwigo%20on%20a%20Raspberry%20Pi,installation%205%20Install%20Piwigo%206%20Optional%20steps.%20)
3. 系统运行起来后，还会缺少 GP 等库，google 后 apt install 就行

Web相册跑起来后，还是很欣慰的，ios App 一开始没搞定，设置好 https 后解决, 截图如下：

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*18eIT6cgwMAAAAAAAAAAAABkARQnAQ)

![](https://ftp.bmp.ovh/imgs/2020/08/f4e312f524b2fc02.png)

如果需要直接读取系统目录上的照片，可以上同步到 galleries 目录中

## 6. 挂PT

**虽无丝竹管弦之盛，一觞一咏，亦足以畅叙幽情
**

<hr></hr>

闲鱼入手的初衷就是省电挂PT，此处终于用到了 docker，实际上不推荐使用，这玩意儿一启动就占用了1G多内存，我买的4G版本，启动各种服务后，几乎是满载状态，爱折腾的建议一步到位，直接买8G内存版本

Docker 安装, 参考文档 [https://shumeipai.nxez.com/2019/05/20/how-to-install-docker-on-your-raspberry-pi.html
](https://shumeipai.nxez.com/2019/05/20/how-to-install-docker-on-your-raspberry-pi.html
) 记得换源

```python
sudo curl -sSL https://get.docker.com | sh

#下载 Docker 图形化界面 portainer
sudo docker pull portainer/portainer
#创建 portainer 容器
sudo docker volume create portainer_data
#运行 portainer
sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

```

安装  transmission 挂PT, 直接撸个镜像. 访问 9091 端口，启动时的参数 -v 就是移动硬盘的挂载位置
```shell
docker run -d \
--restart=always \
--name transmission \
-v /home/pi/torrents:/to_download \
-v /home/pi/download:/output \
-p 9091:9091 \
-p 51413:51413 \
jaymoulin/transmission
```

有 docker 镜像的应用，非常省心，不用Debug，不用调试依赖，无痛启动，另外这个镜像有Bug，登录web页面时，将用户名密码删掉后直接登录

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*l1ljRYzTWy0AAAAAAAAAAABkARQnAQ)

## 7. 开启smb服务

**固知一死生为虚诞，齐彭殇为妄作**

<hr></hr>

挂PT下片后，开启smb服务，电视、手机、电脑等就可以直接看片，速度还行，我连的wifi，也能播放蓝光，就是发热量挺大

执行 
```shell
sudo apt-get install samba samba-common-bin
```

修改配置文件，追加下面的内容，path 指向的是 pt 下载目录

```shell
[share]
   path = /media/pi/PDisk/transmission
   valid users = pi
   browseable = yes
   public = yes
   writable = yes
   guest ok = yes
   read only = no
```
然后重启下 smb

```
sudo service smbd restart
```

如果需要外网访问，445端口与139端口开一下端口映射，手机和电脑都可以直接访问，随时随地无忧看片，不过没啥意义，速度上不来。

建议只在局域网内访问，下面是PC与手机端访问的截图

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*odULRoE7qvEAAAAAAAAAAABkARQnAQ)
![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*Bc8GTaR3fTAAAAAAAAAAAABkARQnAQ)
![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*mj1eTJKrGIkAAAAAAAAAAABkARQnAQ)


## 8. 安装树莓派监控

**向之所欣，俯仰之间，已为陈迹，犹不能不以之兴怀，况修短随化，终期于尽**
<hr></hr>

不想通过top命令查看硬件指标，可以使用netdata做监控，[参考安装文档](https://github.com/netdata/netdata)，效果图如下，十分炫酷，就是很消耗资源，启动后就占用了 700M内存

![](https://gw.alipayobjects.com/mdn/rms_456118/afts/img/A*IgC6QI7Bv0kAAAAAAAAAAABkARQnAQ)

启动命令 <code>sudo /usr/sbin/netdata</code>, 后面会直接用硬件LED屏来显示树莓派状态, NetData 光炫酷了, 内存捉急.

## 9. 安装https

**故列叙时人，录其所述，虽世殊事异，所以兴怀，其致一也。后之览者，亦将有感于斯文。**

<hr></hr>

最后一个篇幅, 留给https配置吧, 手机端相册 piwigo与 seafile 都用得到, 我直接使用的 acme.sh, 非常方便, [参考官方Wiki](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

用acme生成证书后, nginx 设置下, 然后restart, 就能直接访问了.

```python

        # SSL configuration
        #
    listen 443 ssl default_server;
    ssl_certificate         /home/pi/cert/love67/cert.pem;
    ssl_certificate_key     /home/pi/cert/love67/key.pem;
```


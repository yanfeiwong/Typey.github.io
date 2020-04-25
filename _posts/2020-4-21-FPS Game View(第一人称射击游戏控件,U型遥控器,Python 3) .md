---
layout: post
title: 第一人称射击游戏控件(U型遥控器,Python 3)
description: "像在手机上玩FPS一样遥控你的设备！"
modified: 2020-04-21
tags: udp python3 U型遥控器
categories: U-Switch.cn
lang: zh
image:
    feature: hdpage.jpg
---
>像在手机上玩FPS游戏一样遥控你的设备！


## 总览

这个控件包含两个内容：

    1.一个返回手机屏幕上x和y方向滑动量的全屏控件
    2.一个简易的图传（我们将在另一篇文章里介绍）
    
<a href="https://yanfeiwong.github.io//u-switch.cn/JPEG图传(U型遥控器,Python-3)/" target="_blank"><font color="red">JPEG图传示例</font></a>

我们可以用它来做什么？

    1.作为电脑上的触控板
    2.像玩FPS游戏一样控制双舵机云台
    3.当然不止以上内容啦..我只提供工具和基本用法，大家自行发挥 ：）

这篇文章将会展示一个通过python脚本控制双舵机云台的示例，其他的示例也将会在未来编写添加。

> 1.2.8 版本后会在按下和弹起时发送“down”和“up”

<figure class="half center">
	<a href="{{ site.url }}/images/u_FPS_cn/接收数据.png"> <img src="{{ site.url }}/images/u_FPS_cn/接收数据.png" alt=""></a>
    <figcaption>接收到的数据示例</figcaption>
</figure>

<figure class="half center">
	<a href="{{ site.url }}/images/u_FPS_cn/9G舵机云台.jpg"> <img src="{{ site.url }}/images/u_FPS_cn/9G舵机云台.jpg" alt=""></a>
    <figcaption>常见的双舵机云台</figcaption>
</figure>

## 通过树莓派控制由两个9g舵机组成的云台示例
>三代树莓派自带的GPIO输出的PWM信号并不稳定，容易照成舵机抖动，这里我们使用第三方的pigpio库来获取更为稳定的PWM信号输出。

在树莓派上安装pigpio：
{% highlight python %}
sudo apt-get install pigpio
{% endhighlight %}

导入用到的库并且在尝试初始化pigpio
{% highlight python %}
import pigpio,os,time
from socket import *

try:
    os.system("sudo pigpiod")
    tim.sleep(2)
except:
    pass
    
{% endhighlight %}

初始化变量
>一般的廉价9g舵机没有任何保护，不建议直接通过树莓派供电，需要根据每个舵机和安装方式测定舵机的最大最小移动范围，防止赌舵照成损失。

{% highlight python %}
#输出pwm信号的两个针脚（对应两个舵机）
s1=27
s2=22
#x，y方向的滑动灵敏度
res_X=1.0
res_Y=1.2
#舵机的初始位置和最大最小移动范围（根据不同舵机修改，实验测得）
s1_m=1240
s2_m=1100
s1_max=2390
s1_min=500
s2_max=2200
s2_min=540
#舵机pwm信号频率
feq=60
#舵机的当前位置=初始位置
ps_1=s1_m
ps_2=s2_m
#初始化
pi = pigpio.pi()
pi.set_PWM_frequency(s1,feq)
pi.set_PWM_frequency(s2,feq)
#set_servo_pulsewidth()函数使我们的舵机的位置变化可以通过无脑的数字叠加实现
pi.set_servo_pulsewidth(s1,s1_m) 
pi.set_servo_pulsewidth(s2,s2_m)
{% endhighlight %}

配置UDP

{% highlight python %}
host = '' #保持不变
port = 126 #对应手机上第二行的端口
bufsize = 1024 
addr = (host,port)
udpServer = socket(AF_INET,SOCK_DGRAM) 
udpServer.bind(addr)
{% endhighlight %}

定义几个基本的舵机运动控制函数

{% highlight python %}
def stop(): #停止输出pwm信号
    pi.set_servo_pulsewidth(s1,0)
    pi.set_servo_pulsewidth(s2,0)
def mid(): #舵机复位
    global ps_1,ps_2
    ps_1=s1_m
    ps_2=s2_m
    pi.set_servo_pulsewidth(s1,s1_m)
    pi.set_servo_pulsewidth(s2,s2_m)
def set_p(s,p,sm,smax): #设置舵机的位置（s舵机号=pwm输出针脚，p设置的位置，sm当前舵机最小位置，smax最大位置）
    if p>=sm and p<=smax:
        pi.set_servo_pulsewidth(s,int(p))
    elif p>smax:
        p=smax
    elif P<sm:
        p=sm
    return p
{% endhighlight %}

接收循环

{% highlight python %}
while 1:
        data,addr = udpServer.recvfrom(bufsize) 
        data=data.decode() 
        if data == "out": #退出
            stop()
            udpServer.close() 
            print("FPS Out")
            break
        elif data == "mid": #复位
            mid()
        else:
            try:
                #print(data)
                data=data.split(";")  #尝试对收到数据进行拆解为x，y方向上的两个值
                data_1=data[0]
                data_2=data[1]
                ps_1=ps_1-int(data_1)*res_X #当前位置=当前位置+-滑动量*灵敏度
                ps_2=ps_2+int(data_2)*res_Y
                ps_1=set_p(s1,ps_1,s1_min,s1_max) #分配位置给舵机
                ps_2=set_p(s2,ps_2,s2_min,s1_max)
            except:
                pass
exit(0)
{% endhighlight %}


## USwitch端的配置
1.和树莓派处于同一个网络环境

2.设置FPSView，ip设置为树莓派的ip，把端口设置为126（如脚本），注意端口可以更改，尽可能保持独立防止占用
<figure class="half center">
	<a href="{{ site.url }}/images/u_FPS_cn/FPS设置.jpg"> <img src="{{ site.url }}/images/u_FPS_cn/FPS设置.jpg" alt=""></a>
    <figcaption>FPSView设置</figcaption>
</figure>



## 本教程到此就结束:)
### APP下载
<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
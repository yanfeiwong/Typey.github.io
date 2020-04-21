---
layout: post
title: JPEG图传(U型遥控器,Python 3)
description: "简单有效的图传"
modified: 2020-04-21
tags: udp python3 U型遥控器
categories: U-Switch.cn
lang: zh
image:
    feature: hdpage.jpg
---
>JPEG格式由于其高效的压缩效率和标准化要求，目前已广泛用于彩色传真、静止图像、电话会议、印刷及新闻图片的传送


## 为什么使用JPEG图传

主要有以下几个原因：

    1.轻巧，JPEG格式的压缩率是目前各种图像文件格式中最高的；
    2.灵活，我们可以调整图像质量（大小）；
    3.通用，大多数设备支持JPEG编码；
    4.实时，我们可以把整个图传流程简化为拍摄，编码，切片，发送，做到拍一张发一张。
    5.简单，不用考虑复杂的视频编码和编码视频照成的延迟。


这篇文章将会讲解图传的设置，并且展示两个用于实时图传的Python脚本，分别使用OpenCV和树莓派原生的PiCamera模块。


## U-Switchd端设置
<figure class="half center">
	<a href="{{ site.url }}/images/p3_u_cn/FPS.jpg"> <img src="{{ site.url }}/images/p3_u_cn/FPS.jpg" alt=""></a>
    <figcaption>图传设置</figcaption>
</figure>

1.现版本（v1.2.7）图传功能包括在FPSView中，前两行用作实现<a href="https://yanfeiwong.github.io//u-switch.cn/FPS-Game-View(第一人称射击游戏控件,U型遥控器,Python-3)/" target="_blank"><font color="red">类似于触控板的功能</font></a>，对于图传无关紧要，可以随意填写。

2.这里最重要的是开启图传，把接收端口设置为9921（如脚本），注意接收端口可以更改，
但是不可以<font color="red">小于四位数</font>，不可以是已经被<font color="red">占用</font>的端口（检查你的会话界面）。

3.根据需求你可以选择把接收的图片拉伸为全屏，或者复制成左右格式配合VR盒子体验一番。

在手机上安装U型遥控器

<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>

## 使用OpenCV实现图传的示例脚本及注释
>v1.2.7版本以后可以向图传端口发送clear来清理手机上显示的图片

{% highlight python %}
#导入必备库
import cv2,math,time
from socket import *

#启用摄像头
cap = cv2.VideoCapture(0)

#设置视频捕捉的分辨率
cap.set(3,690)
cap.set(4,360)

#UDP配置
host  = '192.168.3.17' #手机上显示的IP
port = 9921 #手机上设置的端口
bufsize = 1024.0
addr = (host,port)
udpClient = socket(AF_INET,SOCK_DGRAM)

while True:
      ret, image_np = cap.read() #拍摄
      cv2.imshow('frame',image_np) #显示图像，可以注释掉
      data=cv2.imencode(".jpg",image_np,[cv2.IMWRITE_JPEG_QUALITY, 60])[1].tobytes() #JPEG编码，60为质量，越低图像越小，越流畅
      cut=int(math.ceil(len(data)/(bufsize))) #计算切片数
      strr="size;"+str(cut)# 通知手机开始接收切片
      udpClient.sendto(strr.encode(),addr)
      for i in range(cut):
          udpClient.sendto(data[i*int(bufsize):(i+1)*int(bufsize)],addr) #切片并且发送
      udpClient.sendto(("end").encode(),addr)#通知手机显示图片
      if cv2.waitKey(25) & 0xFF == 27: #退出
        udpClient.sendto(("clear").encode(),addr) #结束，清理手机上显示的图片,V1.2.7以后版本可用
        cv2.destroyAllWindows() #关闭显示图像的窗口
        cap.release() #释放摄像头
{% endhighlight %}

## 通过PiCamera模块实现图传的示例脚本及注释
>使用树莓派原生的PiCamera模块前请确保已经在raspi-config中启用摄像头

{% highlight python %}
import math,time,sys
from picamera import PiCamera
from socket import *
from io import BytesIO

#UDP配置
host  = '你手机的ip'
port = 9921 #手机上设置的端口
bufsize = 1024.0
addr = (host,port)
udpClient = socket(AF_INET,SOCK_DGRAM)

#相机配置
res_x=640  #照片分辨率
res_y=480
framerate=40 #拍摄帧率，低帧率会模糊
iso=400   #拍摄iso
jpeg_quality=20 #照片质量，太大会卡顿

with PiCamera() as camera:
    #配置相机
    camera.resolution=(res_x,res_y)
    camera.framerate=framerate
    camera.iso=iso
    #准备内存stream
    stream=BytesIO()
    #处理每一张照片（foo）
    for foo in camera.capture_continuous(stream,"jpeg",quality=jpeg_quality,use_video_port=True):#使用占用视频端口的连拍模式
        size=sys.getsizeof(stream) #内存写入文件的大小
        stream.seek(0) #指针到stream的0位置
        cut=int(math.ceil(size)/(bufsize)) #计算切多少片
        strr="size;"+str(cut) 
        udpClient.sendto(strr.encode(),addr) #通知手机开始接收一张图片
        for i in range(cut): #循环发送每一个切片
            d=stream.read(int(bufsize))
            udpClient.sendto(d,addr)
        udpClient.sendto(("end").encode(),addr) #通知手机接受完成显示图片
        stream.seek(0)
        stream.truncate() #内存流刷新

#这个脚本可能有一些问题，会在图片末尾有黑边
{% endhighlight %}

## 本教程到此就结束:)
### APP下载
<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
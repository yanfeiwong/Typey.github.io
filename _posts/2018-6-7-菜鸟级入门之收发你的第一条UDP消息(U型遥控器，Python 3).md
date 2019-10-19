---
layout: post
title: 菜鸟级入门之收发你的第一条UDP消息(U型遥控器，Python 3)
description: "菜鸟级的入门，大神误入！"
modified: 2018-6-7
tags: udp python3 U型遥控器
categories: U-Switch.cn
lang: zh
image:
    feature: hdpage.jpg
---

菜鸟级的入门，大神误入！



## 为什么选择UDP

> 在选择使用协议的时候，选择UDP必须要谨慎。在网络质量令人十分不满意的环境下，UDP协议数据包丢失会比较严重。但是由于UDP的特性：它不属于连接型协议，因而具有资源消耗小，处理速度快的优点，所以通常音频、视频和普通数据在传送时使用UDP较多，因为它们即使偶尔丢失一两个数据包，也不会对接收结果产生太大影响。比如我们聊天用的ICQ和QQ就是使用的UDP协议。 ———摘自《百度百科》
 
#### 然而在实际使用之中我们会发现UPD比TCP方便很多，UDP没有TCP的握手、确认、重传、拥塞控制等机制，发送数据之前不需要建立连接，方便快捷，对实时应用很有用。

## 准备工作
1. 安装Python 3
   1. 如果机器是64位的话就下载64位的，以后学习深度学，习人工智能之类的时候会用到，官网会默认给你x86的
   2. 安装时勾选“Add Python to PATH”，以后也会方便很多
2. 给你的设备设置一个静态IP（防止自己写好脚本，保存好文件以后，IP地址突然变了）
   1. 百度会给你很多种设置静态IP的方法，这里就不一一详述了
   2. 推荐在路由器里通过DHCP设置
3. 在手机上安装U型遥控器

<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-4098168680602409"
     data-ad-slot="1784191902"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

## 接收你的第一条UDP消息
### Python 3 简单接收代码

{% highlight python %}
from socket import * #导入我们要用到的库
host = '' #监听所有端口
port = 123 #用来接收消息的端口
bufsize = 1024 #buf大小，暂时不管
addr = (host,port)
udpServer = socket(AF_INET,SOCK_DGRAM) #通讯类别，对这两个参数感兴趣的可以自己查一下，我后面也会介绍
udpServer.bind(addr)#开始监听
while 1: #死循环
    data,addr = udpServer.recvfrom(bufsize) #接收data
    data=data.decode() #收到的data是bytes类型这里编码成utf8
    if data == "exit": #收到exit以后关闭端口退出程序
        udpServer.close() #不关闭端口的话这个端口在关闭idle以前再用不了了
        print("已退出")
        exit(0) 
    else:
        print(data)
        #if data==..... do .... 干你想干的
{% endhighlight %}

#### 上面的代码理论上可以在任何有网络权限的Python3上执行

#### *如果不是通过接收exit消息来关闭程序的话可能会出现端口占用
#### 提供一条Linux平台的粗暴解决端口占用的方法
{% highlight python %}
import os
os.system("sudo kill -9 $(lsof -ti udp:被占用的端口号)")
#这条代码会直接关掉正使用这个端口的程序，即使你不知道占用它的程序是什么（可能需要安装lsof）
{% endhighlight %}
#### 运行写好的脚本
#### 然后打开我们的U型遥控器手机端
<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/Screenshot_20180607-234249.jpg"> <img src="{{ site.url }}/images/p1_u_cn/Screenshot_20180607-234249.jpg" alt=""></a>
</figure>
##### 第一行添我们刚刚写好代码的那台机子的IP地址
#####    *以Windows为例 快捷键 win+r 打开 cmd 命令窗口，输入ipconfig，其中的IPv4地址就是我们需要的
#####    *以Linux为例 终端输入 ifconfig，返回结果 wlan0 里的 inet addr
##### 第二行添刚刚代码里的port（123）
##### 第三行添用来在手机上接收消息的端口，暂时并没有人给我们发消息，随意添一个数字
##### 下面显示的手机当前的IP为0.0.0.0是因为并没有打开wifi，打开wifi以后重新打开这个页面，记下这个ip我们后面要用
#### 确认所有数据填写无误以后，发送我们的第一条消息吧！（不要忘记发送exit来退出）
<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/01.jpg"> <img src="{{ site.url }}/images/p1_u_cn/01.jpg" alt=""></a>
	<figcaption>执行结果</figcaption>
</figure>
### Python 3 简单发送代码
{% highlight python %}
from socket import *
host  = '192.168.31.119' # 这是手机上刚刚显示的那个灰色的ip
port = 9999 #接口选择大于3位数的，然后填写在手机端的第三栏（手机端其余不变）
bufsize = 1024
addr = (host,port)
udpClient = socket(AF_INET,SOCK_DGRAM)
while 1:
	data=input()#等待输入要发送的消息（输入完回车确认）
	if data=="":
		pass
	else:
		data = data.encode()
		udpClient.sendto(data,addr)
		print("send over")
{% endhighlight %}

#### 上面的代码理论上可以在任何有网络权限的Python3上执行

> *实际使用时可以把发送部分定义成函数，在不方便使用显示器获取输出结果时，它就派上用场了

<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/02.jpg"> <img src="{{ site.url }}/images/p1_u_cn/02.jpg" alt=""></a>
	<figcaption>执行结果</figcaption>
</figure>
## 菜鸟级教程到此就结束啦，很快更新下一篇 :)
### APP下载
<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
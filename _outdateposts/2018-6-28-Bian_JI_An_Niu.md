---
layout: post
title: 手动编辑你的U型遥控器按钮(U型遥控器)
description: "手动编辑你的按钮"
modified: 2018-6-28
tags: udp python3 U型遥控器
categories: U-Switch.cn
lang: zh
image:
    feature: hdpage.jpg
---

手动编辑你的按钮


> 很抱歉目前版本的U型遥控器并不支持在应用里编辑已经保存好的按钮，建议直接删掉然后重新添加，但是我们也可以通过手动修改文件的形式来编辑你已经保存好的按钮，这里做一个超级简易的说明。

## 准备工作
1. 一个可以编辑文件的文件管理器(这里以ES为例)
2. U型遥控器保存的文件在存储空间的“/USwitch/def/”目录下
2. 修改导致的错误或者应用程序无法正常启动，可以直接删除修改过的文件来解决


## 我们以一个消息按钮为例

按钮配置文件的名称为“obutt.1.shut.us”,obutt是按钮类型(消息按钮)，1是每个按钮的编号，注意这个编号尽量不要修改，否则可能会引起消息发送的紊乱，shut是按钮的名称会显示在按钮上

#### 用文本编辑器打开这个配置文件

<figure class="half center">
	<a href="{{ site.url }}/images/ed_u/Screenshot.jpg"> <img src="{{ site.url }}/images/ed_u/Screenshot.jpg" alt=""></a>
	<figcaption>用文本编辑器打开</figcaption>
</figure>

#### 1. 前四行是按钮的位置，按钮左,上,右,下四个方向(顺时针)的边框在屏幕上的像素位置，目前你只可以通过修改它来修改按钮的位置，但是不能修改按钮的大小==..
#### 2. 第五行是接收端的ip
#### 3. 第六行是接收端的端口
#### 4. 第七行是这个按钮要发送的消息
#### 5. 其他类型的按钮和这个基本类似
#### 6. 目前手动编辑可以自定义的功能很少╮(╯▽╰)╭，但是如果大家喜欢，我会更新的(ง •̀_•́)ง！

# 简易说明完
超级简易有没有....软件还在进一步制作，到时候大家应该就不需要这个了吧(ˉ▽￣～)
### APP下载
<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
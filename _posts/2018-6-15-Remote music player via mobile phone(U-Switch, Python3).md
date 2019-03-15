---
layout: post
title: Remote music player via mobile phone(U-Switch, Python3)
description: "DIY a mobile remote music player controller!"
modified: 2018-6-15
tags: udp python3 U-Switch
categories: U-Switch.en
lang: en
image:
    feature: hdpage.jpg
---

DIY a mobile remote music player controller!

## Preparation
1.Prepare a music folder and create our script inside it:
<figure class="half center">
	<a href="{{ site.url }}/images/p2_u_cn/01-en.png"> <img src="{{ site.url }}/images/p2_u_cn/01-en.png" alt=""></a>
	<figcaption>Music folder</figcaption>
</figure>

2.Install pygame library(A very useful library):
{% highlight python %}
pip install pygame
{% endhighlight %}

3.Install U-Switch on your phone :)

<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({
          google_ad_client: "ca-pub-4098168680602409",
          enable_page_level_ads: true
     });
</script>

## Directly on the script and comments
{% highlight python %}
import os
from pygame import mixer # Import our music player from pygame :)
from socket import *


host = ''
phoneip = 'Ip of your phone'
port = 123	# The device receiving port will fill in the phone
sendport = 9999 # Phone receiving port
bufsize = 1024
addr = (host,port)
addr1 = (phoneip,sendport)
udpClient = socket(AF_INET,SOCK_DGRAM) 
udpServer = socket(AF_INET,SOCK_DGRAM)
udpServer.bind(addr)



def findmus(): # Find the music in the current directory, mp3 and flac format, aac does not support by mixer
    L=[]
    l=os.listdir()
    for f in l:
        if f.find(".mp3")==len(f)-4 or f.find(".flac")==len(f)-5:
            L.append(f)  
    return(L)

def play(x): # Play function to prevent crashes due to inability to load
    try:
        mixer.music.load(x)
        mixer.music.play()
        return 0
    except:
        return 1


def send(sdata): # Function to send a message to a mobile phone
    sdata = sdata.encode()
    udpClient.sendto(sdata,addr1)

def Is_Int(s): # Determine if str can be converted to int
    try: 
        int(s)
        return True
    except ValueError:
        return False

L=findmus() # Create an empty list for all found music
mixer.init() 
nowplaying=0 # Used to locate the currently playing song
print("Player started")
print("Find the following songs：")
n=0
for i in L: #Print a song list
    n=n+1
    print(str(n)+"."+i)

while 1: # You can customize the processing of the received command
    data,addr = udpServer.recvfrom(bufsize)
    data=data.decode()
    if data=="exit":
        udpServer.close()
        mixer.quit()
        exit(0)
        
    elif data=="song list":
        n=0
        for i in L:
            n=n+1
            send(str(n)+"."+i)
            
    elif Is_Int(data):
        if play(L[int(data)-1])==0:
            nowplaying=int(data)-1
            send("Now Playing:"+L[nowplaying])
        
    elif data=="play":
        try:
            mixer.music.play()
        except:
            play(L[nowplaying])
            send("Now Playing:"+L[nowplaying])

    elif data=="pause":
        mixer.music.pause()

    elif data=="stop":
        mixer.music.stop()

    elif data=="next":
        nowplaying=nowplaying+1
        if nowplaying>len(L):
            nowplaying=0
        play(L[nowplaying])
        send("Now Playing:"+L[nowplaying])

    elif data=="last":
        nowplaying=nowplaying-1
        if nowplaying<0:
            nowplaying=len(L)
        play(L[nowplaying])
        send("Now Playing:"+L[nowplaying])
    else:
        send("Sorry, only these instructions are now supported:song list，number on demand，play，pause，stop，next，last,exit")

{% endhighlight %}

Changehe the IP address and port, it should be able to run on any platform.
<figure class="half center">
	<a href="{{ site.url }}/images/p2_u_cn/02-en.png"> <img src="{{ site.url }}/images/p2_u_cn/02-en.png" alt=""></a>
	<a href="{{ site.url }}/images/p2_u_cn/03-en.jpg"> <img src="{{ site.url }}/images/p2_u_cn/03-en.jpg" alt=""></a>
	<figcaption>Result</figcaption>
</figure>
Of course, the most convenient thing is to make the commands into buttons :)
<figure class="half center">
	<a href="{{ site.url }}/images/p2_u_cn/04-en.jpg"> <img src="{{ site.url }}/images/p2_u_cn/04-en.jpg" alt=""></a>
	<figcaption>Music Remote</figcaption>
</figure>

### U-Switch now is available on:

<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
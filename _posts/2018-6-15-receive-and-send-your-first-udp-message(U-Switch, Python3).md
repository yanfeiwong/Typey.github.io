---
layout: post
title: Receive and send your first udp message(U-Switch, Python3)
description: "For beginners:)"
modified: 2018-6-15
tags: udp python3 U-Switch
categories: U-Switch.en
lang: en
image:
    feature: hdpage.jpg
---

For beginners:)

## Why choose UDP?
#### In actual use, we will find that UPD is much more convenient than TCP. UDP does not have TCP handshake, acknowledgement, retransmission, and congestion control mechanisms. It does not need to establish a connection before sending data. It is convenient and fast, useful for real-time applications.
## Preparation:
1. Install Python 3
2. Set a static IP for your device(In case you just finished scripts and the IP address suddenly changed,Recommended through DHCP)
3. Install U-Switch on your phone :)

<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({
          google_ad_client: "ca-pub-4098168680602409",
          enable_page_level_ads: true
     });
</script>

## Receive your first UDP message
### Simple Receive Code for Python 3

{% highlight python %}
from socket import * 
host = '' # Listen on all ports
port = 123 # The port used to receive the message
bufsize = 1024 
addr = (host,port)
udpServer = socket(AF_INET,SOCK_DGRAM)
udpServer.bind(addr)#Start listening
while 1:
    data,addr = udpServer.recvfrom(bufsize) ## Receive data
	data=data.decode() # The data received is a bytes type encoded here as utf8
    if data == "exit": # Close port and exit program after receiving exit
        udpServer.close() # If you forget to close the port, could try this in idle
		print("Exited")
        exit(0) 
    else:
        print(data)
        #if data==..... do .... do whatever you want 
{% endhighlight %}

#### The above code can theoretically be executed on any Python 3 with network permission
#### * If the program closed not by receiving the message "exit", then may appear a occupied port issue,when you run it again
#### Here is a slution on Linux:
{% highlight python %}
import os
os.system("sudo kill -9 $(lsof -ti udp:PORT_NUMBER)")
#This code will directly shut down the program that is using this port, even if you don't know what the program is :)
{% endhighlight %}
#### Run the script
#### Open U-Switch,create a Chat
<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/Screenshot_20180607-234249.jpg"> <img src="{{ site.url }}/images/p1_u_cn/Screenshot_20180607-234249.jpg" alt=""></a>
</figure>
##### The first line adds the IP address of the device we just wrote the code
##### The second line adds port"123" in the code
##### The third line adds the port used to receive messages on the phone. No one gonna to sent us a message for now :) Add a number at random.
##### The current IP of the phone shown below is 0.0.0.0 because wifi is closed. Open the wifi and re-open this page. Write down the ip we will use it later.
#### After confirming that all data is correct, send us our first message!
<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/01.jpg"> <img src="{{ site.url }}/images/p1_u_cn/01.jpg" alt=""></a>
	<figcaption>Results</figcaption>
</figure>
### Python 3 Simple Sending Code
{% highlight python %}
from socket import *
host  = '192.168.31.119' # This is the gray ip you just showed on your phone.
port = 9999 # Receive port should be greater than 3 digits, and then fill in the third column of the mobile terminal (the rest unchanged)
bufsize = 1024
addr = (host,port)
udpClient = socket(AF_INET,SOCK_DGRAM)
while 1:
	data=input()# Wait for the message input
	if data=="":
		pass
	else:
		data = data.encode()
		udpClient.sendto(data,addr)
		print("send over")
{% endhighlight %}
#### The above code can theoretically be executed on any Python 3 with network permission also
> *The sending part can be defined as a function, when it is not convenient to use the display to get the output, it comes in handy.
<figure class="half center">
	<a href="{{ site.url }}/images/p1_u_cn/02.jpg"> <img src="{{ site.url }}/images/p1_u_cn/02.jpg" alt=""></a>
	<figcaption>Results</figcaption>
</figure>
### Download app from here:
<a href='https://play.google.com/store/apps/details?id=com.typey.tool.uswitch&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img alt='Get it on Google Play' src='https://play.google.com/intl/en_us/badges/images/generic/en_badge_web_generic.png' height="83" width="215"/></a>
<a href='https://www.coolapk.com/apk/188229'><img alt='Get it on CoolApk' src='{{ site.url }}/images/coolan.png' height="80" width="150"/></a>
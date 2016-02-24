---
layout:     post
title:      Nas on linux(bananapi)
date:       2016-02-15 14:00:18
summary:    bananapi
categories: Linux,Bananapi
---


##Ready
1.Banana Pi

2.Install ArchLinux(or other linux) 

3.setting sshd_config


--

##1.DDNS
1.apt-get update

2.install dig for ddns

    apt-get install dnsutils

3.-->此处添加ddns python代码

4.add rc.local

     vi /etc/rc.local
     
##2.mount device




{% highlight C %}
	diskutil unmountDisk /dev/diskN
{% endhighlight %} 





  


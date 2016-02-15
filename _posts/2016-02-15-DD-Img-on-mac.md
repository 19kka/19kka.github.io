---
layout:     post
title:      Writing Images on MacOS
date:       2016-02-15 14:00:18
summary:    use dd
categories: Linux
---

##1.Be root or your are root(admin)

    su


##2.unmount Disk
{% highlight C %}
	diskutil list
{% endhighlight %}

####find your disk then unmount it

looks like 
/dev/disk0
...........TYPE NAME...

/dev/disk1
........XXX...XXX

{% highlight C %}
	diskutil unmountDisk /dev/diskN
{% endhighlight %} 

you will see:
xxxxxx diskN was successful

##3.Use the 'dd' 

    sudo dd if=xxx.iso of=/dev/sdX bs=4096



  


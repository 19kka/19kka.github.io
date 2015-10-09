---
layout:     post
title:      python tips
date:       2015-10-09 9:00:18
summary:    关于python的一些实用技巧
categories: python
---

汇总一些自己遇到的python的小技巧。

###python内嵌shell函数
使用  [subpro module](https://docs.python.org/2/library/subprocess.html) 库

{% highlight python lineno %}
	from subprocess import call
	call(["ls","-l"])
{% endhighlight %}


   


之后补上。
  


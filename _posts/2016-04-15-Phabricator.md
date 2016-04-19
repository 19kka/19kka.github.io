---
layout:     post
title:      Phabricator on Mac
date:       2016-04-15 14:00:18
summary:    code review 
categories: phabricator,Mac,Git
---


#Phabricator on Mac
--
本文简述了在Mac上安装Phabricator的基本步骤
######引言
Phabricator 是一个LAMP(Linux,Apache,MySQL,PHP) 应用。需要:

 * 一个Linux类系统  
 * 一些基本的系统管理员能力  
 * Git  
 * PHP,MySQL


####安装准备
>Apache,nginx,或者其他的一个web服务器  
>PHP:5.2以上  
>MySQL:建议5.5以上

这些安装在本文中不涉及，自行Google
###安装
####创建文件夹以及安装依赖

    $ mkdir ~/WWW/tools
    $ cd ~/WWW/tools
    $ git clone https://github.com/phacility/phabricator.git
    $ git clone https://github.com/phacility/libphutil.git
    $ git clone https://github.com/phacility/arcanist.git


####添加Arcanist到你的环境变量路径

1.   vi **~/.profile** or **~/.bashrc**
2.   add **export PATH="$HOME/WWW/tools/arcanist/bin:$PATH"**
3.   save **:wq**
4.   使配置生效 **source ~/.profile*/**
5.   测试 **arc help**

####配置Phabricator

#####配置apache配置文件 

     vi xxxxx/apache2/httpd.conf
     
    
```json
<VirtualHost *>
    # Change this to the domain which points to your host.
    ServerName phabricator.local.nfl.com   

    # Change this to the path where you put 'phabricator' when you checked it
    # out from GitHub when following the Installation Guide.
    #
    # Make sure you include "/webroot" at the end!
    DocumentRoot /Users/admin/WWW/tools/phabricator/webroot  

    RewriteEngine on
    RewriteRule ^/rsrc/(.*)     -                       [L,QSA]
    RewriteRule ^/favicon.ico   -                       [L,QSA]
    RewriteRule ^(.*)$          /index.php?__path__=$1  [B,L,QSA]
</VirtualHost>





## Allow access to the entire WWW directory
<Directory "/Users/admin/WWW">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
ServerName localhost
```

*注意把DocumentRoot 修改为自己的路径 （我的是admin)*  

#####将域名解析到本机 :  

* <span style="background-color:AliceBlue ">127.0.0.1  phabricator.local.nfl.com</span> 添加到hosts <span style="background-color:AliceBlue ">vi /private/etc/hosts</span>  

之后重启 apache <span style="background-color:AliceBlue ">$ sudo apachectl -k restart</span>

###进行最后设置
用浏览器打开 [phabricator.local.nfl.com](http://phabricator.local.nfl.com/) 如果之前的操作都没有问题，这个网页会被打开。
![image](https://raw.githubusercontent.com/19kka/19kka.github.io/master/images/pha_01.png)
嗯，是这样的图就对了。然后你需要做一些配置，解决它告诉你的issue（一般有20多个）。安装到此结束。
     
---
1. [https://secure.phabricator.com/book/phabricator/article/installation_guide/](https://secure.phabricator.com/book/phabricator/article/installation_guide/)
2. [https://secure.phabricator.com/book/phabricator/article/arcanist_mac_os_x/](https://secure.phabricator.com/book/phabricator/article/arcanist_mac_os_x/)

---
layout:     post
title:      ssh-copy-id
date:       2016-05-26 14:03:18
summary:    Esay to copy .pub 
categories: Liunx
---

###ssh-copy-id

####Mac : 

>brew install ssh-copy-id

  
####Synopsis:
ssh-copy-id [-i filename] [user@]machine  

####Example:

>ssh-copy-id -i mykey.pub root@1.2.3.4  


If the -i option is given then the identity file (defaults to ~/.ssh/id_rsa.pub) is used, regardless of whether there are any keys in your ssh-agent. 

Once it has one or more fingerprints (by whatever means) it uses ssh to append them to ~/.ssh/authorized_keys on the remote machine (creating the file, and directory, if necessary)



####References:  
[ssh-copy-id](http://linux.die.net/man/1/ssh-copy-id)

[ssh-agent wiki](https://en.wikipedia.org/wiki/Ssh-agent)

[ssh-add from freebsd](https://www.freebsd.org/cgi/man.cgi?query=ssh-add&sektion=1)
---
layout: post
title:  "远程查看tensorboard"
date:   2019-07-31 11:22:00 
tags:   工具
---

<h3>通过以下命令来登录服务器:</h3>  

> `ssh -L 16006:127.0.0.1:6006 account@server.address`  
> 其中16006为本地端口，6006为远程的端口，即tensorboard默认使用的端口，account为登录账号，server.address为服务器地址。  
> 该方式通过ssh来实现远程端口到本地端口的数据转发。 

<h3>然后在服务器运行tensorboard:</h3>  

> `tensorboard --logdir=path-to-log`  
> 若需要更换tensorboard默认端口，可以通过参数`--port=num`的方式来更换  

<h3>最后在本地访问:</h3>   

> `http://127.0.0.1:16006/`

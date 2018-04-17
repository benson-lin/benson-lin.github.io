---
title: "Linux SSH免密码登录"
layout: post
date: 2018-04-17
tags:
- Linux
categories: LINUX
blog: true
excerpt:  SSH免密码登录包括SSH登录，SCP远程传输。可以很方便的在shell脚本中进行命令操作而不需要输入密码
---

# ssh免密码登录

1. 服务器B生成公钥私钥对：`ssh-keygen -t rsa   `，`~/.ssh/`目录下多了两个文件`id_rsa`和`id_rsa.pub`，分别是私钥和公钥

2. 把公钥`id_rsa.pub`发送到服务器A，并记得重命名，以免把别的公钥覆盖。

   ` scp id_rsa.pub root@192.168.16.117:~/.ssh/id_rsa.pub_temp`

3. 将`id_rsa`公钥的内容添加到.ssh目录下的authorized_keys文件，记得以**追加**的方式添加，以免将别的公钥覆盖，若文件不存在则新建。

   `cat id_rsa.pub_temp >> authorized_keys`
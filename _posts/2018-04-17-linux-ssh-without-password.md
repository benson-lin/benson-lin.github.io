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

1. 服务器A生成公钥私钥对：`ssh-keygen -t rsa   `，`~/.ssh/`目录下多了两个文件`id_rsa`和`id_rsa.pub`，分别是私钥和公钥

2. 把公钥`id_rsa.pub`发送到服务器B，并记得重命名，以免把别的公钥覆盖。

   ` scp id_rsa.pub root@192.168.16.117:~/.ssh/id_rsa.pub_temp`

3. 将`id_rsa.pub.temp`公钥的内容添加到B服务器的.ssh目录下的authorized_keys文件，记得以**追加**的方式添加，以免将别的公钥覆盖，若文件不存在则新建。

   `cat id_rsa.pub_temp >> authorized_keys`

4. A登录到B就可以免密码登录了



### 失败分析

如果失败，有可能是以下原因：

**1、权限问题**

.ssh目录，以及/home/当前用户 需要700权限，参考以下操作调整

sudo chmod 700 ~/.ssh

sudo chmod 700 /home/当前用户

.ssh目录下的authorized_keys文件需要600或644权限，参考以下操作调整

sudo chmod 600 ~/.ssh/authorized_keys

**2、StrictModes问题**

编辑`vim /etc/ssh/sshd_config` ，找到`\#StrictModes yes，`改成`StrictModes no`

修改后重启`/etc/init.d/ssh restart`

修改为no,默认为yes.如果不修改，用key登陆是出现`server refused our key`
(如果StrictModes为yes必需保证存放公钥的文件夹的拥有与登陆用户名是相同的.“StrictModes”设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权。
这通常是必要的，因为新手经常会把自己的目录和文件设成任何人都有写权限。)

**3、 MaxAuthTries 1**

错误提示: Too many authentication failures
默认 1，用户输入密码错误的次数，如果输入密码错误系统将自动锁定N分钟才能解锁重新登录



如果还不行，可以用ssh -vvv 目标机器ip 查看详情，或者到/var/log/secure查看ssh登录日志






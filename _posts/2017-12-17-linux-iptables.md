---
title: "Linux iptables 使用"
layout: post
date: 2017-11-06
tags:
- Linux
categories: LINUX
blog: true
excerpt: 包括表、链、命令行的使用等等
---

# iptables

## 1. 表链

链：

- PREROUTING 的规则可以存在于：raw表，mangle表，nat表。
- INPUT 的规则可以存在于：mangle表，filter表，（centos7中还有nat表，centos6中没有）。
- FORWARD 的规则可以存在于：mangle表，filter表。
- OUTPUT 的规则可以存在于：raw表mangle表，nat表，filter表。





报文流向：

- 网卡-->PREROUTING-->是本机-->INPUT->WEB服务-->OUPUT-->POSTROUTING
- 网卡-->PREROUTING-->不是本机-->FORWARD-->POSTROUTING
- 其中WEB服务位于用户空间，INIPUT等位于内核空间


- 到本机某进程的报文：PREROUTING --> INPUT

- 由本机转发的报文：PREROUTING --> FORWARD --> POSTROUTING

- 由本机的某进程发出报文（通常为响应报文）：OUTPUT --> POSTROUTING

  ​

表：

- filter表：负责过滤功能，防火墙；内核模块：iptables_filter

- nat表：network address translation，网络地址转换功能；内核模块：iptable_nat

- mangle表：拆解报文，做出修改，并重新封装 的功能；iptable_mangle

- raw表：关闭nat表上启用的连接追踪机制；iptable_raw

- 每个链的优先级顺序为：raw --> mangle --> nat --> filter，有些没有则忽略

  ​

表链关系：

- 某些"链"中注定不会包含"某类规则"


- raw 表中的规则可以被哪些链使用：PREROUTING，OUTPUT
- mangle 表中的规则可以被哪些链使用：PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING
- nat  表中的规则可以被哪些链使用：PREROUTING，OUTPUT，POSTROUTING（centos7中还有INPUT，centos6中没有）
- filter 表中的规则可以被哪些链使用：INPUT，FORWARD，OUTPUT



## 2. 命令行

### 2.1 查看规则 iptables （-t filter） -nvxL （INPUT）--line

```shell
[root@VM_172_210_centos ~]# iptables -t filter -nvxL INPUT --line 
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num      pkts      bytes target     prot opt in     out     source               destination
1      144783 16664741 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22
2        2998   165732 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:3306
3      297726 38483535 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable

```



### 2.2 增删保存iptables

#### 增加规则

先指定表(-t，默认是filter表)，再指定是插入到某个链最前(-I INPUT)，还是插入到某个序号(-I INPUT 2)，还是到最后(-A)，接着指定匹配条件(如-s指定)最后指定动作(-j)，动作一般是(ACCEPT/REJECT/DROP)

`iptables -t fitler -I INPUT -s 10.139.189.152 -j DROP`

`iptables -t fitler -I INPUT 2 -s 10.139.189.152 -j REJECT`

`iptables -t filter -A INPUT -s 10.139.189.152 -j ACCEPT `

#### 删除规则

先查看`iptables -t filter -nvL INPUT --line`查看需要删除的规则编号，再使用`iptables -t filter -D INPUT id`删除

清空某个表下的所有规则(慎用)：`iptables -t filter -F INPUT`

可以更具体到某条规则：`iptables -t filter -D INPUT -s 192.168.1.146 -j DROP`

####保存规则

centos 6 直接通过`serivce iptables save`就可以了，保存在`/etc/sysconfig/iptables`

centos 7，通过下面配置后，就可以使用`service iptables restart `

```
#配置好yum源以后安装iptables-service
# yum install -y iptables-services
#停止firewalld
# systemctl stop firewalld
#禁止firewalld自动启动
# systemctl disable firewalld
#启动iptables
# systemctl start iptables
#将iptables设置为开机自动启动，以后即可通过iptables-service控制iptables服务
# systemctl enable iptables
```

### 2.3 匹配条件

当有多个规则时，需多个条件同时满足时才匹配规则

最基础的：`iptables -t fitler -I INPUT -s 10.139.189.152 -j DROP`或`iptables -t filter -I OUTPUT -d 192.168.1.111 -j DROP`

指定多个IP：`iptables -t fitler -I INPUT -s 10.139.189.152,10.139.189.153 -j DROP`

指定网段：`iptables -t fitler -I INPUT -s 10.139.189.152/16 -j DROP`

使用取反：`iptables -t fitler -I INPUT ! -s 10.139.189.152 -j DROP`

匹配目标地址：`iptables -t fitler -I INPUT -s 10.139.189.152 -d  10.139.189.153 -j DROP`

匹配协议：`iptables -t fitler -I INPUT -s 10.139.189.152 -p tcp -j DROP`

匹配网卡：`iptables -t fitler -I INPUT -i eth4 -p icmp -j DROP`或`iptables -t filter -I OUTPUT -p icmp ! -o eth4 -j DROP`

匹配目标端口：`iptables -t fitler -I INPUT -s 10.139.189.152 -p tcp -m tcp --dport	22 -j DROP`(-p与-m扩展模块一致时-m可省略)；指定多个连续的端口：`iptables -t fitler -I INPUT -s 10.139.189.152 -p tcp --dport	22:25 -j DROP`，匹配源端口：`iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport :22 -j REJECT`

一次多个端口：`iptables -t filter -I OUTPUT -d 192.168.1.146 -p udp -m multiport --sports 137,138 -j REJECT` 或`iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 22,80 -j REJECT`







 


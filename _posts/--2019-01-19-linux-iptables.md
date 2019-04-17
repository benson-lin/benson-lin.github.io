---
title: "linux iptables使用 "
layout: post
date: 2019-01-19
tag:
- Linux
categories: LINUX
blog: true
---

## iptables使用

安装：

```bash
yum install iptables-services 
```

最常用：

(注意drop之前一定要先允许登录，这里因为我是要限制eth0是外网网卡，只对外暴露10002端口，本身外网就是不能登录的，那行注释掉了，内网eth1的没做限制)

```bash
iptables -nvL INPUT --line-numbers
iptables -F
iptables -A INPUT -p icmp -j  ACCEPT
iptables -A INPUT -i lo -j  ACCEPT
iptables -A INPUT -i eth0 -p tcp -m state --state RELATED,ESTABLISHED -j ACCEPT
#iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,36000 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp -m multiport --dports 10002 -m iprange --src-range 219.143.197.160-219.143.197.161 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp  -s 0.0.0.0/0  -j  DROP
iptables-save  >  /etc/sysconfig/iptables
```
(许多博客说设置默认drop, `iptables  -P  INPUT  DROP`，我感觉不太好，因为设置后一旦不小心清空列表了，就连内网都连不上了)

单个操作：

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT 123.207.88.17 -j ACCEPT
iptables -A INPUT 192.168.122.0/24 -j ACCEPT
#iptables -A INPUT -m iprange --src-range 192.168.122.2-192.168.122.34 -j ACCEPT
#iptables -A INPUT -m iprange --dest-range 8.8.8.2-8.8.8.22 -j DROP
```

删除：

```bash
iptables -t filter -D  INPUT 5
```

插入某一位置：

```bash
iptables -I INPUT 5 -i eth0 -p tcp -m multiport --dports 10002 -m iprange --src-range 123.207.88.17 -j ACCEPT
```




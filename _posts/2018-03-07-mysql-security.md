---
title: "Mysql安全"
layout: post
date: 2018-03-07
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: 安全相关内容，根据mysql官方文档翻译并归纳
---

# Mysql安全

安全指南方针：

1. 不要给任何人(除了root账号外)访问`mysql.user`表
2. 学习mysql访问权限是如何工作的，使用grant和revoke控制权限，不要授予非必要之外的其他权限，不要授权权限给所有的主机host

   - 尝试`mysql -u root`能不能直接登录，如果没有应该设置密码

   - 使用`show grants`查看其他用户拥有哪些权限，使用`revoke`移除非必要的权限
3. 不要在数据库中使用明文，应当使用hash函数对密码进行计算，同时为了防止被字典强破，可以加上盐进行计算保存(hash(hash(password)+salt)
4. 不要使用纯英文单词作为密码。可以选择一个句子中的每个单词的第一个单词作为密码
5. 将数据库放到防火墙之下，可以减少至少50%的漏洞利用。
   - 试着从网络中使用如`	nmap`的工具扫描端口，mysql默认使用3306端口，这个端口不应该被不信任的主机访问。一个简单的检查方法是，尝试从远程机器使用`telnet server_host 3306`命令来测试，其中server_host为mysql所在的主机名或IP地址
   - 如果telnet挂起或连接被拒绝，说明端口阻塞了，正是符合预期的。如果获得连接和收到了包，说明端口开放了，应该从防火墙和路由器上进行关闭。除非有好的理由开放这个端口。
6. 不要信任任何由用户输入的数据，应当使用适当的编程防御措施。
7. 不要在网络上传输未加密的数据，使用诸如SSL或SSH的加密协议。mysql支持内部的ssl连接，另一种方法是使用SSH端口转发来创建一个加密的通信(压缩)隧道。
8. 学习使用tcpdump和strings工具，在大多数情况下,您可以检查MySQL数据流是否加密通过发出一个命令如下。`tcpdump -l -i eth0 -w - src or dst port 3306 | strings`
---
title: "Linux常用命令"
layout: post
date: 2017-04-12
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: Linux常用命令，持续更新
---

## 查看端口是否被占用

```linux
netstat -anp| grep 8080

lsof -i:8080
```
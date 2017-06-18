---
title: "linux常见问题解决"
layout: post
date: 2017-06-17
tags:
- Linux
categories: LINUX
blog: true
excerpt: 玩linux中遇到的问题：乱码问题，持续更新
---

## 常见问题

1. 展示乱码：修改/etc/profile，在最后加上export LANG=zh_CN.UTF-8。保存后source /etc/profile生效。同时修改终端仿真程序的编码，如secureCRT。可以通过$LANG查看当前的系统语言编码
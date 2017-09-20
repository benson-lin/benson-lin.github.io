---
title: "Jquery选择器"
layout: post
date: 2017-08-31
tags:
- JAVASCRIPT
categories: JAVASCRIPT
blog: true
excerpt: 找父节点，兄弟节点等
---


## Jquery选择器

- jQuery.parent(expr)           //找父元素
- jQuery.parents(expr)          //找到所有祖先元素，不限于父元素
- jQuery.children(expr)        //查找所有子元素，只会找到直接的孩子节点，不会返回所有子孙
- jQuery.contents()            //查找下面的所有内容，包括节点和文本。
- jQuery.prev()                //查找上一个兄弟节点，不是所有的兄弟节点
- jQuery.prevAll()             //查找所有之前的兄弟节点
- jQuery.next()                //查找下一个兄弟节点，不是所有的兄弟节点
- jQuery.nextAll()             //查找所有之后的兄弟节点
- jQuery.siblings()            //查找兄弟节点，不分前后
- jQuery.find(expr)            
- //跟jQuery.filter(expr)完全不一样，jQuery.filter(expr)是从初始的, jQuery对象集合中筛选出一部分，而jQuery.find()的返回结果，不会有初始集中筛选出一部分，比如：
$("p").find("span")是从元素开始找，等于$("p span")


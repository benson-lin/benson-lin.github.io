---
title: "CSS备忘--内边距，边框，外边框"
layout: post
date: 2017-02-17
tags:
- CSS
categories: CSS
blog: true
excerpt: CSS备忘--内边距，边框，外边框
---

# CSS备忘--内边距，边框，外边框


**宽度和高度**：widht/height, 不能应用到行内非替换元素，如a标签，浏览器会忽略。

**外边距**：

auto：左右为auto则会居中；左右其中一个为auto，则auto值设置为元素刚好充满父元素的内容大小

百分比：会相对父元素进行设置外边距

margin: 15px ==> margin: 15px 15px 15px 15px;
margin: 15px 10px ==> margin: 15px 10px 15px 10px;
margin: 15px 10px 12px ==> margin: 15px 10px 12px 10px;

对行内非替换元素的上下外边距不发挥作用，因为行内非替换元素的外边距不会改变一个元素的行高，左右外边距则有影响；但对行内替换元素则有影响，会影响其行高

**边框**：宽度或粗细，样式或外观，颜色

元素的背景是内容、内边距和边框区的背景

border-style: none(默认)/hidden/solid/dashed(虚线)/dotted(点线)

可以通过border-style: solid dash none dashed；类似margin的方式设置四个方向的边框。对单边样式，提供了border-top-style, border-right-style, border-bottom-style, border-left-style;

border-width: 边框宽度；thin, medium, thick或者自己的值；同样的有border-top-width等设置单边宽度，如果边框样式是none或hidden，则不管border-width设置为多大，最终都是为0

border-color: <color>或transpareng; 也有border-top-color等；

透明边框，有些情况想创建不可见边框，也就是transparent，创建有宽度的不可见边框

border: 1px solid #000;对边框进行统一处理，顺序不重要；也有border-bottom等属性；如果缺少一个值，会按默认值填充，这个要注意

特殊：不论行内元素的边框指定怎样的宽度，元素的行高都不会改变，但是是允许的，会被绘制出来

**内边距**：padding

是百分数时，相对于父元素的width计算

内边距对行内元素两侧可以发挥作用，但是不会应用到各行的左右两边；对替换元素也是如此；对行内元素的高也能改变，对行高本身没有改变。


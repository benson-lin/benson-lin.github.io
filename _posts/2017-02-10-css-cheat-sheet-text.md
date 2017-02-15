---
title: "CSS备忘--文本属性"
layout: post
date: 2017-02-10
tags:
- CSS
categories: CSS
blog: true
excerpt: CSS备忘--文本属性
---

# CSS备忘--文本属性


text-indent: 缩进首行文本，只能用于块级元素，对行内元素和图像之类的无效；如果设置为百分比表示相对父元素的宽度的百分之几进行缩进；对行内元素，可以进行外边距或左内边距的调整达到一样的效果；这个属性会继承，而且是继承计算后的值，使用时需注意；

text-align：水平对齐；有left，right，center；只能应有块级元素。

line-height：行高，文本行基线之间的距离；用来增加文本行直接的垂坠间隔。(line-height-font-size)/2得到文本上方和下方的像素大小。当一个块级元素继承了另一个元素的line-height属性，可能会变得复杂，这是line-height应当设置为继承值而不是计算值；如果设为inherit，和普通的继承没有不同，只是特殊性和不设置是不同的。

vertical-align：垂直对齐文本；有top,buttom,middle等

word-spacing：字母间隔

letter-spacing：字符间隔

text-decoration：文本装饰；有none, underline, line-through；无继承性，表示文本上的装饰**线**与父元素的颜色相同（父元素经过了子元素的文本），即使后代元素本身有其他颜色，因为没有继承性，后代元素的text-decoration默认是none，此时无法去除子元素的下划线；如果希望子元素有自己的下划线颜色，需要将子元素的text-decoration显式声明，然后再定义颜色


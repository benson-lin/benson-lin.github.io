---
title: "css备忘"
layout: post
date: 2017-02-06
tags:
- CSS
categories: CSS
blog: true
excerpt: CSS备忘
---

# CSS备忘

## 选择器

.warning {}：所有class="warning"的元素，相当于*.warning
p.warning {}：所有class="warning"的p元素
p.warning {}.urgent:所有class既包含warning也包含urgent的p元素，缺一不可
\#warning {}: 所有id="warning"的元素


h1{class} {}: 有class属性的h1元素
a{href="abc"} {}: href属性为"abc"的a元素
p[class="warning urgent"] {}: 如果class本身有空格的，需要用[]，和p.warning.urgent不同的是，这里是字符串完全匹配，也就是说有顺序的
p[class~="warning"] {}:和p.warning一样的效果，也就是必须是空格隔开的某一个；存在的意义在于不只是用在class属性中，否则和p.warning就一样了
p[foo^="warning"] {}: foo属性以bar开头的
p[foo$="warning"] {}: foo属性以bar结尾的
p[foo*="warning"] {}: foo属性包含bar的；只需要模糊有匹配就可以了


p span {}: 后代选择器，p元素下的后代为span的元素
ul ol ul em {}:　ul下的ol元素,ul下的ul元素,ul下的em元素，不管嵌套多深，只要子元素，孙元素等都能作用到；
div span, p b {}: div下的所有span元素，p下的b元素
h1 > strong{}: 子选择器，只选择到h1下一层子元素的strong标签，不再向下查找
table.summary td > p{}: 后代选择器和子选择器结合使用
h1 + p {}: 兄弟选择器，选择紧接在h1后出现的所有p标签，且h1和p具有相同的父元素


a:visited {}: 已访问过的链接，伪元素的选择器必须放到最后，下同
a:link {}: 默认的
a:hover {}: 移到链接上不点击时
a:active {}: 点住链接不放手
a.external:link {}: 指定更小范围的
input:focus {}: 可以接受键盘输入时的样式
a#copyright:link {}: 可以用在id上

.one p:first-child {}: 选择class="one"下的第一个p元素
p:first-letter {}: 首字母样式
p:first-line {}:第一行样式
h2:before(content: "}}";color:silver): h2元素前添加两个大括号
h2:after:在元素后面在内容
---
title: "CSS备忘"
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

**类选择器和id选择器**：
.warning {}：所有class="warning"的元素，相当于*.warning
p.warning {}：所有class="warning"的p元素
p.warning {}.urgent:所有class既包含warning也包含urgent的p元素，缺一不可
\#warning {}: 所有id="warning"的元素

**属性选择器**：
h1{class} {}: 有class属性的h1元素
a{href="abc"} {}: href属性为"abc"的a元素
p[class="warning urgent"] {}: 如果class本身有空格的，需要用[]，和p.warning.urgent不同的是，这里是字符串完全匹配，也就是说有顺序的
p[class~="warning"] {}:和p.warning一样的效果，也就是必须是空格隔开的某一个；存在的意义在于不只是用在class属性中，否则和p.warning就一样了
p[foo^="warning"] {}: foo属性以bar开头的
p[foo$="warning"] {}: foo属性以bar结尾的
p[foo*="warning"] {}: foo属性包含bar的；只需要模糊有匹配就可以了

**后代选择器**：
p span {}:p元素下的后代为span的元素
ul ol ul em {}:　ul下的ol元素,ul下的ul元素,ul下的em元素，不管嵌套多深，只要子元素，孙元素等都能作用到；
div span, p b {}: div下的所有span元素，p下的b元素


**后代选择器和兄弟选择器**：
h1 > strong{}: 子选择器，只选择到h1下一层子元素的strong标签，不再向下查找
table.summary td > p{}: 后代选择器和子选择器结合使用
h1 + p {}: 兄弟选择器，选择紧接在h1后出现的所有p标签，且h1和p具有相同的父元素

**伪类选择器和伪元素选择器**：
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
h2:after(content: "}}";color:silver): 在元素后面在内容

## 特殊性：specificity

ID选择器: 0，1，0，0
类选择器、属性选择器、伪类选择器：0，0，1，0
标签选择器、伪元素选择器：0，0，0，1

对>, +等会忽略，只按实际写的为准；比如 p > strong 为0,0,0,2

**通配选择器的特殊性**： 0,0,0,0，任何css样式都会替换掉通配选择器的样式；这个容易理解，因为 .cla 相当于 \*.cla, 而 .cla 是0，0，1，0, 所以 * 本身就是0，0，0，0

**ID和属性选择器的特殊性**：ID选择器和指定id属性的属性选择器在特殊性上有所不同： html > body table tr[id="totals"] td ul >li {} 为 0,0,1,7，而 li#answer {} 为 0，1，0，1；对指定id属性的属性选择器其实只是贡献了0,0,1,0

**内联样式特殊性**：第一个0是为内联样式声明保留的，也就是说优先级最高，特殊性最高


## 重要性声明

p.dark {color: #333 !important; background: white;}

标志为!important的声明没有特殊的特殊性值，而是与非重要性声明分开考虑，当与非重要性声明冲突时，总是重要声明胜出；或者是特殊性失效了。


## 继承

基于继承机制，样式不仅可以应用到指定元素，还可以应用到它的后代元素。

不能继承的属性：边框属性border，框模型属性如外边距、内边距、背景、边框

**继承值没有特殊性**，连0特殊性都没有；0特殊性比无特殊性强，下面的例子1em元素将显示为gray；而例子2的链接则可能被浏览器默认的样式表替代，显示成蓝色，这时要对a标签进行特殊出来才行

```css
/** example1 */
<h1 id="page-title">1111<em>222222</em></h1>
\* {color: gray;}
h1#page-title {color: black;}

/** example2 */
<div id="toolbar">11111<a href="#">2222</a></div>
#toolbar {color: red}; 
/** 需要添加这个 */
#toolbor a:link {color: white;}

```


## 层叠

当特殊性相等时，以哪一个为准；通过继承和特殊性完成

```css
h1 {color: red};
h1 {color: blue};
```


**层叠规则**：
- 权重大小：读者的重要声明>创作人员的重要声明>创作人员的正常声明>读者的正常声明>用户代理声明（按权重和来源排序）
- 当权重相同时，才按特殊性进行处理（按特殊性排序）
- 当特殊性还一样时，再按文件顺序，以后面那个为准（按顺序排序）


CSS中的样式一共有三种来源：**创作人员、读者和用户代理**：
- 首先，创作人员（author's style）样式应该是我们最熟悉的，如果你是一个前端开发者，那么你写的那些样式就叫做创作人员样式。
- 然后是用户代理样式（agent's style），用户代理也就是我们通常所说的浏览器（IE、Firefox等等），这些浏览器会提供一些默认的样式，比如IE浏览器中，一个纯粹由html代码构成的网页里，我们会发现超链接会带有一个蓝色的前景色，这其实就用户代理样式，借用一些插件我们可以方便的查看这些默认样式（比如Firefox中的Web+developer）
- 最后，也是最容易被我们忽略的，读者样式（reader's style）。所谓读者自然就是浏览网页的用户，有些时候这些用户里可能会有人不满意网页的配色，或者字体大小，这时候他们就是通过浏览器提供的接口为网站添加读者样式。


因为有上面的规则，所以在对a标签进行处理时是根据 link-visited-hover-active(LVHA-CSS2推荐的顺序,VLHA也可以)的顺序进行声明，因为都是0,0,1,0,具有相同的权重、来源和特殊性；因此与元素匹配的最后一个选择才会胜出

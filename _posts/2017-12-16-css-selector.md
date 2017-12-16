---
title: "CSS选择器"
layout: post
date: 2017-12-16
tags:
- CSS
categories: CSS
blog: true
excerpt: 无
---

# CSS选择器分类

## 基本选择器

- 通配符选择器(\*)：如`* {}`或`.demo * {}`
- 元素选择器(.className)：如`.demo {}`或`p.items {color: red;}`(选中p标签上class为items的元素)或`.important.items {}`(选中同时包含了"important"和"items"两个类的元素)
- ID选择器：如`#demo {}`
- 后代选择器E F：如`p span{}`选择p下所有的span标签，包括所有后代元素
- 子元素选择器E > F：如`ul > li {background: green;color: yellow;}`只能选中子元素，孙元素无法选择
- 相邻兄弟元素E + F：如 `li.active + li {color:red;}` 被选中时，所有兄弟li元素
- 通用兄弟选择器E ~ F：如`li.active ~ li {}` ，和E+F不同的是，只会选择active的li后面的所有li元素，前面的不会选中
- 群组选择器(sel1,sel2...)：如`.first, .second {}`选择多个

## 属性选择器

- E[attr]：如`.demo[id]`找到类名是demo且有id属性的；如`demo[href][id]`找到含有href和id属性的值
- E[attr="value"]：找到attr属性是value的；如果有多个class如有`class=".a .b"`，如`.demo[class="a b"]`才能拿到。
- E[attr~="value"]：只要有其中一个就匹配`.demo a[title~="website"]`。**多个属性值要用空格隔开**
- E[attr^="value"]：属性以value开头的
- E[attr$="value"]：属性以value结尾的
- E[attr*="value"]：属性包含value子串的
- E[attr|="value"]：属性以value开头或者以value-开头的元素

## 伪类选择器

- 动态伪类：第一种是我们在链接中常看到的锚点伪类，如":link",":visited";另外一种被称作用户行为伪类，如“:hover”,":active"和":focus"。添加的顺序应该为Link--visited--hover--active。

```
a:link {color:gray;}/*链接没有被访问时前景色为灰色*/
a:visited{color:yellow;}/*链接被访问过后前景色为黄色*/
a:hover{color:green;}/*鼠标悬浮在链接上时前景色为绿色*/
a:active{color:blue;}/*鼠标点中激活链接那一下前景色为蓝色*/
```

- UI元素状态伪类：":enabled",":disabled",":checked"。前两个针对type="text"的元素，后一种针对type="radio"或type="checkbox"的元素
- CSS3的:nth选择器：

```
:first-child选择某个元素的第一个子元素；如 .li:first-child {}
:last-child选择某个元素的最后一个子元素；
:nth-child()选择某个元素的一个或多个特定的子元素；
  :nth-child(n);/*参数是n,n从0开始计算，加到结束为止*/
  :nth-child(n*length)/*n的倍数选择，n从0开始算*/，如 li:nth-child(2n)，选择2*0(无),2*1,2*2的元素
  :nth-child(n+length);/*选择大于等于length后面的元素*/，如 li:nth-child(n+5)从第五个li开始选择 
  :nth-child(-n+length)/*选择小于等于length前面的元素*/，如 li:nth-child(4n+1)每个四个选一个
  :nth-child(n*length+1);/*表示隔几选一*/
:nth-last-child(n)选择某个元素的一个或多个特定的子元素，从最后一个子元素开始算；选择倒数第几个

:nth-of-type()选择指定的元素；p:nth-of-type(even) {}每隔一个p元素就有不同的样式
:nth-last-of-type()选择指定的元素，从元素的最后一个开始计算；
:first-of-type选择一个上级元素下的第一个同类子元素；
:last-of-type选择一个上级元素的最后一个同类子元素；

:only-child选择的元素是它的父元素的唯一一个子元素；
:only-of-type选择一个元素是它的上级元素的唯一一个相同类型的子元素；
:empty选择的元素里面没有任何内容。
```

- 否定选择器(not)：`input:not([type="submit"]) {border: 1px solid red;}`
- 伪元素：`:first-line,::first-letter,::before,::after,::selection`。::first-line选择元素的第一行；::first-letter选择文本块的第一个字母；::before和::after这两个主要用来给元素的前面或后面插入内容，这两个常用"content"配合使用，见过最多的就是清除浮动。::selection用来改变浏览网页选中文的默认效果

```
  .clearfix:before,
  .clearfix:after {
    content: ".";
    display: block;
    height: 0;
    visibility: hidden;
  }
  .clearfix:after {clear: both;}
  .clearfix {zoom: 1;}
```




---
title: "CSS备忘--颜色和背景"
layout: post
date: 2017-02-17
tags:
- CSS
categories: CSS
blog: true
excerpt: CSS备忘--颜色和背景
---

# CSS备忘--颜色和背景

css 可以为任何元素设置前景色和背景色；

## 前景

前景是元素的文本，还包括元素周围的边框：可以通过color属性，也可以使用边框属性设置;

需要注意的是color会作用在边框上，如果需要覆盖掉边框的颜色，需要在border加入颜色；

下面的边框颜色是以 #0f0 为准的，而字体将显示为 #00f, 如果去除 #0f0，边框将显示为 #00f

```css
		.div1 {
			color: #00f;
			border: 1px solid #0f0;
		}
```

利用边框可以影响图片的前景色。虽然图像本身无法被color影响，但是可以影响图片周围的边框，也就是说color或border-color都可以做的；

color 具有继承性，如果body设为红色，会把默认没有其他样式的文本设置为红色。但是如果浏览器有预定义的颜色，会使得无法继承到color，如table的color值；此时我们需要特殊处理，如 body, table, td, th {color: red;}

## 背景

元素背景区包括前景之下直到边框外边界的所有空间，内容框，内边距都是背景的部分，而边框在背景之上；也就是说边框如果设置为transparent且设置了背景色，那么边框将显示为背景色, 下面的10px的边框将显示为#ccc

```css
		.div1 {
			border: 10px solid transparent;
			background-color: #ccc;
		}
```

**背景色**：background-color, 默认是transparent，无继承性

**背景图像**：background-image, 默认是none，默认是不断的复制同一张图片直到铺满，也就是repeat。

用法为 body { background-image: url(bg.gif); }

将图片放到指定的背景颜色上，如果完全平铺gif，jpeg等不透明的图像，背景颜色将失效；而对alpha通道的图片如png类型，图像和背景色可能会结合。

**有方向的重复**：background-repeat: repeat | repeat-x | repeat-y | no-repeat | inherit; 可以用在元素上，产生很好看的效果

用法为 body { background-image: url(bg.gif); background-repeat: repeat-x;}

repeat如果不想在左上角开始，可以通过背景定位改变初始位置，如从中间开始重复

**背景定位**：background-position: center | left | right | top | bottom， 还可以设置长度和百分比

如果设置负值，会将图片移出背景区，如只需要显示图片的某部分，就可以这样做

**关联**：background-attachment： scroll | fixed | inherit；默认是scroll；

如果文档很长，而背景图片在中间，可能需要滚动几页才能看到背景图片，此时使用fix，原图像不会随文档滚动，其次，原图像的放置有可视区的大小决定，而不是包含该图像的元素大小决定，实现了固定背景功能；

使用fixed，将相对于可视区定位，而不是相对于包含图像的元素定位

可以将全部都写在 **background** 属性中，无需规定顺序，如果缺少值会填充对应的默认值，而且没有必不可少的值，只要至少出现一个值就行
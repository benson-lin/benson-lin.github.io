---
title: "JavaScript 的event.stopPropagation()和event.preventDefault() "
layout: post
date: 2018-05-19
tag:
- JavaScript
categories: JAVASCRIPT
blog: true
excerpt: event.stopPropagation()阻止事件冒泡，event.preventDefault()阻止浏览器默认行为。
---

# event.stopPropagation()和event.preventDefault() 

**1.event.stopPropagation()方法**

这是阻止事件的冒泡方法，不让事件向documen上蔓延，但是默认事件任然会执行，当你掉用这个方法的时候，如果点击一个连接，这个连接仍然会被打开，

**2.event.preventDefault()方法**

这是阻止默认事件的方法，调用此方法是，连接不会被打开，但是会发生冒泡，冒泡会传递到上一层的父元素；

**3.return false  ；**

这个方法比较暴力，他会同事阻止事件冒泡也会阻止默认事件；写上此代码，连接不会被打开，事件也不会传递到上一层的父元素；可以理解为return false就等于同时调用了event.stopPropagation()和event.preventDefault()



## 例子

### 1. 默认行为

在div上click事件，当点击a标签跳转时，也会打出log

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<meta charset="utf-8">
	<style type="text/css">
		.div {
			height: 200px;
			width: 200px;
		}
		.div a {
			display: block;
			background: red;
			width: 100px;
			height: 100px;
		}
	</style>
	<script type="text/javascript" src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
	<script type="text/javascript">
		
		$('.div').click(function(){
			console.log('click a');
		});


	</script>
</head>
<body>
	<div class="div">
		<a href="http://www.google.com" target="_blank">打开谷歌</a>
	</div>
</body>
</html>
```

### 2. 阻止冒泡

在a标签加上`event.stopPropagation();`后点击a不再打印log，但是也没会正常跳转

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<meta charset="utf-8">
	<style type="text/css">
		.div {
			height: 200px;
			width: 200px;
		}
		.div a {
			display: block;
			background: red;
			width: 100px;
			height: 100px;
		}
	</style>
	<script type="text/javascript" src="https://code.jquery.com/jquery-3.3.1.min.js"></script>

</head>
<body>
	<div class="div">
		<a href="http://www.google.com" target="_blank">打开谷歌</a>
	</div>
	<script type="text/javascript">
		$('.div').click(function(){
			console.log('click a');
		});

		$('.div a').click(function(event){
			event.stopPropagation();
		});
	</script>
</body>
</html>
```

### 3.阻止默认时间 


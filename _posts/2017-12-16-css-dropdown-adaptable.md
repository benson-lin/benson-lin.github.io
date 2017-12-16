---
title: "CSS实现自适应和下拉菜单"
layout: post
date: 2017-12-16
tags:
- CSS
categories: CSS
blog: true
excerpt: 无
---

# CSS实现自适应和下拉菜单

### 重点

下拉菜单功能：

1. 利用checkbox和label以及css的`checked`属性实现下来菜单功能
2. 需要用`clear:both`对菜单进行清除浮动，不能用`overflow:hidden;`否则菜单部分会被隐藏
3. 使用绝对定位和相对定位放置下拉菜单位置



自适应功能：

1. 在head中加入`<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">`
2. 使用`@media screen and (min-width: 750px) {}`标签
3. 使用百分比实现自适应，对图片也使用
4. 可以对图片进行`max-width`进行像素最大值控制，防止图片随着屏幕一直变大



## 效果如下

#### 当屏幕较大时

![](/assets/images/2017-12-16-css-dropdown-adaptable-01.jpg)

#### 当屏幕较小时

![](/assets/images/2017-12-16-css-dropdown-adaptable-02.jpg)

### 代码

* 文件地址：`https://github.com/benson-lin/Learning-JavaScript-CSS/tree/master/note-css/CSS实现自适应和下拉菜单`

```html
<!DOCTYPE html>
<head>
	<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	<title>And the winner isn't…</title>

	<script type="text/javascript" src="jquery-3.2.1.js"></script>
	<script type="text/javascript">
		$(function(){
			$('#nav-2-nav a').click(function(){

				console.log('11111');
			});
		});
	</script>
	<style type="text/css">
		#wrapper {
			width: 80%;
			margin: 0 auto;
			max-width: 1000px;
			overflow: hidden;
		}

		#wrapper #header #nav-1 ul{
			padding:0;
		}
		#wrapper #header #nav-1 ul >  li{
			display: inline-block;
			margin-right: 2%
		}
		#wrapper #header #nav-1 ul > li a{
			text-decoration: none;
		}

		#wrapper #header #nav-2 ul{
			padding: 0px;
			margin: 0;
		}
		#wrapper #header #nav-2 ul > li{
			text-decoration: none;
			list-style: none;
			background: #1ABC9C;
			padding: 8px 5px;
		}
		#wrapper #header #nav-2 ul > li:hover{
			background: #CCCCCC;

		}

		#wrapper #header #nav-2 ul > li a{
			text-decoration: none;
		}

		#wrapper #main #sidebar{
  			float: left;
  			width: 35%
		}
		#wrapper img {
			max-width: 300px;
			max-height: 300px;
		}

		#wrapper #main #sidebar img{
			max-width: 40%;
		}

		#wrapper #main {
			overflow: hidden;
		}

		#wrapper #content{
  			float: right;
  			width: 65%
		}

		@media screen and (min-width: 750px) {
			h1 {
				font-size: 2em;
			}
			#wrapper #header #logo {
				font-size: 1.5em;
			}
			#wrapper #header #nav-1 ul > li{
				font-size: 1.2em;
			}
			#wrapper #header #nav-2 {
				display: none;
			}
		}
		@media screen and (max-width: 750px) {
			h1 {
				font-size: 1.5em;
			}
			#wrapper #header #logo {
				font-size: 1em;
			}

			#wrapper #header #nav-1 ul > li{
				font-size: 0.8em;
			}

			#wrapper #header #nav-2{
				display: none;
			}
		}
		@media screen and (max-width: 520px) {
			#clear {
				clear: both;
			}
			#wrapper #header #nav-1{
				display: none;
			}
			* {
				font-size: 0.8em;
			}
			h1 {
				font-size: 1em;
			}
			#wrapper #header #logo {
				display: inline-block;
				float: left;
			}
			#wrapper #header #nav-2 {
				display: inline-block;
				min-width: 120px;
				float: right;
				position:relative;
			}
			#wrapper #header #nav-2 #nav-2-menu  {
				display: none;
			}
			#wrapper #header #nav-2 #nav-2-menu ~ ul{
				display: none;
			}
			#wrapper #header #nav-2 #nav-2-menu:checked ~ ul{
				position: absolute;
				display: block;
				top: 20px;
				float: left;
				background-color: red;
			}

		}
	</style>
</head>
<body>
	<div id="wrapper">
		<!-- the header and navigation -->
		<div id="header">
			<div id="logo">And the winner is<span>n't...</span></div>
			<div id="nav-2">
				<input id="nav-2-menu" type="checkbox">
				<label for="nav-2-menu" class="">Menu</label>
				<ul id="nav-2-nav">
						<li><a href="#">Why?</a></li>
						<li><a href="#">Synopsis</a></li>
						<li><a href="#">Stills/Photos</a></li>
						<li><a href="#">Videos/clips</a></li>
						<li><a href="#">Quotes</a></li>
						<li><a href="#">Quiz</a></li>
				</ul>
			</div>

			<div id="nav-1">
				<ul>
					<li><a href="#">Why?</a></li>
					<li><a href="#">Synopsis</a></li>
					<li><a href="#">Stills/Photos</a></li>
					<li><a href="#">Videos/clips</a></li>
					<li><a href="#">Quotes</a></li>
					<li><a href="#">Quiz</a></li>
				</ul>
			</div>
		</div>
		<div id="clear"></div>
		<div id="main">
			<!-- the content -->
			<div id="content">
				<h1>Every year <span>when I watch the Oscars I'm annoyed...</span></h1>
				<p>that films like King Kong, Moulin Rouge and Munich get the statue whilst
					the real cinematic heroes lose out. Not very Hollywood is it?</p>
					<p>We're here to put things right. </p>
					<a href="#">these should have won &raquo;</a>
			</div>
			<!-- the sidebar -->
			<div id="sidebar">
				<div class="sideBlock unSung">
					<h4>Unsung heroes...</h4>
					<a href="#"><img src="1.jpg" alt="Midnight Run" /></a>
					<a href="#"><img class="sideImage" src="1.jpg" alt="Wyatt
						Earp" /></a>
				</div>
				<div class="sideBlock overHyped">
					<h4>Overhyped nonsense...</h4>
					<a href="#"><img src="1.jpg" alt="Moulin Rouge" /></a>
					<a href="#"><img src="1.jpg" alt="King Kong" /></a>
				</div>
			</div>
		</div>
	
		<!-- the footer -->
		<div id="footer">
			<p>Note: our opinion is absolutely correct. You are wrong, even if you think
				you are right. That's a fact. Deal with it.</p>
		</div>
	</div>
</body>
</html>

```






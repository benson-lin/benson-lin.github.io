---
title: "JavaScript 变量提升"
layout: post
date: 2016-11-29
tag:
- JavaScript
categories: JAVASCRIPT
blog: true
excerpt: JavaScript Scoping and Hoisting。JavaScript变量提升问题，变量提升使JavaScript与其他编程语言有很大的不同。
---

# JavaScript 变量提升

先上结论：JavaScript没有块级作用域，也就是说，在C语言或者Java中看到的if语句后面用大括号括起来形成的局部作用域没了,在if语句外部也能访问内部的变量。对JavaScript而言，只有全局作用域和函数作用域(包括匿名函数)，除非使用函数表达式方式。


一个典型的问题：

```js
var a = 1;
function test(){
	alert(a);
	var a = 2;
}
```

没学过C语言的人可能认为是1，因为按顺序来说，下面的a是在后面定义的，alert中的a应当是在函数外部的，但是JavaScript并不能以这种思维去考虑。

JavaScript变量提升指的是在变量的作用域内，会将变量的定义提升到这个作用域的最顶层。也就是说上面的代码会变成：

```js
var a = 1;
function test(){
	var a;
	alert(a);
	a = 2;
}
```

因此会弹出 undefined。


对**函数**而言，比如下面这段代码是能够成功调用的，因为是函数声明方式的。


```js
function test(){ 
	foo(); 
	function foo(){ 
		alert("foo"); 
	} 
} 
test(); 
```

下面这个就不行，将报未定义错误`Script snippet #1:2 Uncaught TypeError: foo is not a function(…)`。

```js
function test(){ 
	foo(); 
	var foo = function foo(){ 
		alert("foo"); 
	} 
} 
test(); 
```


## var 有无的区别

没有加var会将变量应用于全局环境，而加上var则其作用域是变量所在的某个作用域(当然如果放在全局环境就是全局作用域)

下面的alert(b)会弹出undefined，从原理上因为变量b的声明被扔到了最上面

```js
alert(b);
if(1){
	var b = 1;
}
```

也就是相当于

```js
var b;
alert(b);
if(1){
	b=1;
}
```

如果将var去掉将报错, 错误为`Uncaught ReferenceError: b is not defined(…)`，而不是undefined。因为b没有定义就使用，因为不能将这个变量提到前面。


```js
alert(b);
if(1){
	b = 1;
}
```

不能变为

```js
//b;
alert(b);
if(1){
	b = 1;
}
```


那为什么一个会显示undefined，而另一个却是报错了,《JavaScript高级程序设计》中举了个很简单的例子：对message，因为进行了定义，所以是弹出undefined，而age则报未定义的错误，这样就比较容易理解了。

```js
var message; //this variable is declared but has a value of undefined
//make sure this variable isn’t declared
//var age
alert(message); //”undefined”
alert(age); //causes an error
```
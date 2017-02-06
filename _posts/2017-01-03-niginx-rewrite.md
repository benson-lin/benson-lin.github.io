---
title: "Nginx rewrite"
layout: post
date: 2016-01-03
tag:
- Nginx
categories: NGINX
blog: true
excerpt: 本博文通过在本地启动三台应用服务器，两台tomcat和一台apache服务器作为nginx服务器的分发机器。意在讲清upstream的简单使用。
---

# Nginx rewrite

在使用nginx配置rewrite中经常会遇到有的地方用last并不能工作，换成break就可以，其中的原理是对于根目录的理解有所区别，按我的测试结果大致是这样的。

location /   
{    
    proxy_pass http://test;    
    alias /home/html/;    
    root /home/html;    
    rewrite "^/a/(.*)\.html$" /1.html last;    
}  
在location / { 配置里：
1、使用root指定源：使用last和break都可以
2、使用proxy_pass指定源：使用last和break都可以
3、使用alias指定源：必须使用last
在location /a/或使用正则的location ~ ^/a/里：
1、使用root指定源：使用last和break都可以
2、使用proxy_pass指定源：使用break和last结果有所区别
3、使用alias指定源：必须使用last
其中区别主要在proxy_pass这个标签上，再看看几个测试结果：

location /    
{    
    root /home/html;    
}    
  
location /a/   
{    
    proxy_pass http://test;    
    rewrite "^/a/(.*)\.html$" /1.html last;    
}  
在这段配置里，使用last访问是可以访问到东西的，不过，它出来的结果是：/home/html/1.html；可我需要的是http://test/1.html？使用break就可以了。

location /   
{    
    root /home/html;   
}    
    
location /a/   
{    
    proxy_pass http://test;    
    rewrite "^/a/(.*)\.html$" /a/1.html last;   
}  
在这段配置里，返回错误，因为last会重新发起请求匹配，所以造成了一个死循环，使用break就可以访问到http://test/a/1.html。
所以，使用last会对server标签重新发起请求，而break就直接使用当前的location中的数据源来访问，要视情况加以使用。一般在非根的location中配置rewrite，都是用的break；而根的location使用last比较好，因为如果配置了fastcgi或代理访问jsp文件的话，在根location下用break是访问不到。测试到rewrite有问题的时候，也不妨把这两者换换试试。
至于使用alias时为什么必须用last，估计是nginx本身就限定了的，怎么尝试break都不能成功。



Rewrite规则
rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如 http://seanlook.com/a/we/index.php?id=1&u=str 只对/a/we/index.php重写。语法rewrite regex replacement [flag];

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。

表明看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里，它们的执行顺序是：

执行server块的rewrite指令
执行location匹配
执行选定的location中的rewrite指令
如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误。


flag标志位
last : 相当于Apache的[L]标记，表示完成rewrite
break : 停止执行当前虚拟主机的后续rewrite指令集
redirect : 返回302临时重定向，地址栏会显示跳转后的地址
permanent : 返回301永久重定向，地址栏会显示跳转后的地址


https://segmentfault.com/a/1190000002797606
---
title: "Nginx location配置"
layout: post
date: 2016-12-17
tag:
- Nginx
categories: NGINX
blog: true
---

# Nginx location配置


**语法规则： location [=|~|~*|^~] /uri/ { … }**

注意：在写这种格式的时候记得一定要加上uri前面和后面的斜杠/，否则可能出现奇奇怪怪的问题；从官网文档看，location 只能放在 server 下。

- = 开头表示精确匹配；但是不支持正则表达式
- ^~ 开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码。
- ~ 开头表示区分大小写的正则匹配
~* 开头表示不区分大小写的正则匹配- 
- !~和!~*分别为区分大小写不匹配及不区分大小写不匹配 的正则
- / 通用匹配，任何请求都会匹配到。
- @ 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
- 空： 匹配任何请求，因为所有请求都是以"/"开始，但是更长字符匹配或者正则表达式匹配会优先匹配



```nginx
 server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location = / {

        }
        location = /login {

        }
        location ^~ /static/ {

        }
        location ~ \.(gif|jpg|png|js|css)$ {

        }
        location ~* \.png$ {

        }
        location !~ \.xhtml$ {

        }
        location !~* \.xhtml$ {

        }
        location / {
            index  index.html index.htm;
        }
    }
```


那么产生的效果如下：

- 访问根目录/， 比如http://localhost/ 将匹配规则A
- 访问 http://localhost/login 将匹配规则B，http://localhost/register 则匹配规则H
- 访问 http://localhost/static/a.html 将匹配规则C
- 访问 http://localhost/a.gif, http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用， 而 http://localhost/static/c.png 则优先匹配到 规则C
- 访问 http://localhost/a.PNG 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。
- 访问 http://localhost/a.xhtml 不会匹配规则F和规则G，http://localhost/a.XHTML不会匹配规则G，因为不区分大小写。规则F，规则G属于排除法，符合匹配规则但是不会匹配到，所以想想看实际应用中哪里会用到。
- 访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候应该是nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。


location 匹配规则优先级：

1. 用uri测试所有的prefix string;
2. 字符串精确匹配到一个带 “=” 号前缀的location，则停止，且使用这个location的配置；
3. 字符串匹配剩下的非正则和非特殊location: 匹配最长prefix string，如果这个最长prefix string带有^~修饰符，使用这个location，停止搜索，否则存储这个最长匹配，进入下一步；
4. 字符串正则匹配，匹配顺序为location在配置文件中出现的顺序。如果匹配到某个正则location，则停止，并使用这个location的配置；否则，使用步骤2中得到的具有最大字符串匹配的location配置。
5. 最后

说了这么多，最重要的是第三和第四点：这两点其实就是说先匹配^~，其他的匹配，如~或者~*匹配了了也先不管，还要先看正则匹配的结果；如果正则匹配了，则使用正则的规则，如果正则没有找到，那么才会用回前面的~或~*匹配的规则。

结论：首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。


实际使用中，至少有三个匹配规则定义，如下：

```nginx
#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
#这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}

# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}

#第三个规则就是通用规则，用来转发动态请求到后端应用服务器
#非静态文件请求就默认是动态请求，自己根据实际把握
#毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了

location / {
    proxy_pass http://tomcat:8080/
}
```
---
title: "Nginx upstream 配置负载均衡"
layout: post
date: 2016-12-21
tag:
- Nginx
categories: NGINX
blog: true
excerpt: 本博文通过在本地启动三台应用服务器，两台tomcat和一台apache服务器作为nginx服务器的分发机器。意在讲清upstream的简单使用。
---

# Nginx upstream 配置负载均衡

ngnix将请求分发到几个不同的服务器上。

两个tomcat分别对应两个端口：一个是8088，一个是8080，作为两个动态服务器；还有一个apache作为静态服务器，对jpg类型的图片从apache获取，端口是9090, 静态服务器下有一个apache的logo图片。下面是截图；

我们这边配置了hosts: `127.0.0.1 www.test.me`

![](/assets/images/2016-12-21-nginx-upstream-01.png)

![](/assets/images/2016-12-21-nginx-upstream-02.png)

![](/assets/images/2016-12-21-nginx-upstream-03.png)


将upstream放到nginx.conf中的http下:配置项最重要是是 proxy_pass 和 upstream。 proxy_pass 的格式是 http://upName, 对应的 upstream 标签的名字，这里的 upName 是 apache-tomcat。 upstream 标签内最简单的就是只配置 sever，因为 tomcat 服务器都放在本地跑，所以 IP 都是 127.0.0.1。 nginx 这里是监听 80 端口(listen 80)。 nginx 启动之后，http://www.test.me 访问，多刷新几次就可以看到切换效果(当然两个tomcat要先启动)。

```nginx

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    #重点
    upstream apache-tomcat{
        server 127.0.0.1:8080;
        server 127.0.0.1:8088;
    }
    upstream apache{
        server 127.0.0.1:9090;
    }

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
            #重点
            proxy_pass http://apache-tomcat;
        }

        location ~* \.(jpg)$ {
            #重点
            proxy_pass http://apache;
        }
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

最后的结果截图如下，这样同样的域名访问就分发到两个服务器了。

![](/assets/images/2016-12-21-nginx-upstream-04.png)

![](/assets/images/2016-12-21-nginx-upstream-05.png)


为了演示静态服务器的使用，我们在两个apache服务器下的index.jsp中都加入`<img src="apache.png">`，因为是png结尾的，所以会走 upstream 为apache对应的服务器。

两个index.jsp就是大致加多了这两个内容：

```html
    <body>
        port: 8088 <!-- 另一个台是:port: 8080 -->
        <img src="apache.jpg">
        <!-- ... -->
	</body>
```


启动 apache 服务器，这时三台应哟个服务器还有niginx服务器都起来了。

修改了配置ngnix.conf之后要记得重启(nginx -s stop关闭再nginx启动)。 再次访问`www.test.me`，刷新多几次，可以看到 apache 服务器上的图片找到了，静态文件成功分发到了 apache 服务器，而动态文件依然是走 tomcat。

![](/assets/images/2016-12-21-nginx-upstream-06.png)

![](/assets/images/2016-12-21-nginx-upstream-07.png)

![](/assets/images/2016-12-21-nginx-upstream-08.png)


## 小结

在真正的生产环境中，我们不可能将服务器和 nginx 放在同一个服务器上，因为这样分发也就没有意义了；

更多的意义是：比如我们有一个域名 bensonlin.me，我们配置的时候将域名映射的 IP 填为 niginx 服务器的地址；对于动态网页，我们可以配置到多台应用服务器，如tomcat服务器(对java而言)，或者php服务器，就像上面一样；对于静态页面，比如js, css, png等文件，我们可以单独配置到静态服务器上，如apache。每台服务器都放到自己的不同的IP上，也就是说上面的127.0.0.1都改成其他的IP。这样的好处是：能够得到很好的容灾，每台服务器也能发挥自己的优势(tomcat 对静态文件性能不是很高，我们就交给 apache 处理)。
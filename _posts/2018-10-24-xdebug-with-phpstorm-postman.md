---
title: "phpstorm使用xdebug(在浏览器/postman中)"
layout: post
date: 2018-10-24
tag:
- Php
categories: PHP
blog: true
---

# 浏览器/postman中使用xdebug

## 前提

1. nginx配置

```
	server {
        listen       80;
        server_name  local.tce.ticket.tss.sng.com;

        access_log  logs/access.log;

        location / {
            root   D:/web-package/nginx/html/tce-ticket-BE/public;
			index  index.php index.html index.htm;
			
			try_files $uri $uri/ /index.php?$query_string;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:10001;  
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  D:/web-package/nginx/html/tce-ticket-BE/public$fastcgi_script_name;
            include        fastcgi_params;
        }

    }
```

2. 启动php

   ```
   start cmd /k "cd ../php-7.1.22 && php-cgi -b 127.0.0.1:10001 -c ./php.ini && pause"
   ```



##  下载和配置debug

1. 首先去`https://xdebug.org/download.php`中下载对应php版本的xdeug扩展，如下为php7.1非线程安全64位版本。对应的，可以在网站上找到。（版本一定要对，如果发现进入首行断点，但是F8进入不了下一行，可能就是因为版本不对）

![](/assets/images/2018-10-24-phpv.jpg)

![](/assets/images/2018-10-24-xdebug.jpg)



2. 下载下来后放到php按照目录下的ext目录下

3. php.ini中配置下面文件

   ```
   [Xdebug]
   zend_extension="D:\web-package\php-7.1.22\ext\php_xdebug-2.6.1-7.1-vc14-nts-x86_64.dll"
   xdebug.idekey=PHPSTORM
   xdebug.remote_enable=true      ;Xdebug允许远程IDE连接
   xdebug.remote_host=127.0.0.1   ;允许连接的zend studio的IP地址
   xdebug.remote_port=9876        ;反向连接zend studio使用的端口
   xdebug.remote_handler=dbgp     ;用于zend studio远程调试的应用层通信协议
   ```



## phpstorm 配置

其他不要动了！防止越配越错！配置完点击phpstom的小电话监听，并打开调试就可以在浏览器访问链接调试了

![](/assets/images/2018-10-24-phpstorm-port.jpg)

![](/assets/images/2018-10-24-phpstorm-servers.jpg)

## 如何使用postman调试 

Cookie中添加xdebug.idekey的值

![](/assets/images/2018-10-24-postman.jpg)








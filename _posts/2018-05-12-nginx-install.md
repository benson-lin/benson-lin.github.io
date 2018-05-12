---
title: "Nginx 安装"
layout: post
date: 2018-05-12
tag:
- Nginx
- Linux
categories: NGINX
blog: true
---

# Nginx 安装

1. 下载pcre,zlib和openssl，并下载nigix安装包

   ```bash
    wget -O /tmp/pcre-8.42.tar.gz  http://jaist.dl.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.gz
   tar -zxcf  pcre-8.42.tar.gz
   cd pcre-8.42.tar.gz
   cd pcre-8.42/  
   ./configure  
   make && make install 
   
   
   wget  -O /tmp/zlib-1.2.11.tar.gz http://zlib.net/zlib-1.2.11.tar.gz
   ./configure
   make && make install 
   
   
   wget -O /tmp/openssl-1.1.1-pre6.tar.gz http://www.openssl.org/source/openssl-1.1.1-pre6.tar.gz
   ./config  # 注意是config,不是configure
   make
   make install
   
   ```

   

2. 安装Niginx

   ```bash
   wget -O /tmp/nginx-1.13.12.tar.gz http://nginx.org/download/nginx-1.13.12.tar.gz
   tar -zxvf nginx-1.13.12.tar.gz
   cd /tmp/nginx-1.13.12
   mkdir /usr/local/services/nginx-1.13.12
   ./configure --prefix=/usr/local/services/nginx-1.13.12/ --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module  --with-openssl=/tmp/openssl-1.1.1-pre6
   make && make install
   
   如果configure报错，类似
   ./configure: error: SSL modules require the OpenSSL library.
   You can either do not enable the modules, or install the OpenSSL library
   into the system, or build the OpenSSL library statically from the source
   with nginx by using --with-openssl=<path> option
   的，则需要指定源代码路径，如
   --with-pcre=/lamp/pcre-8.32 \ 
   --with-zlib=/lamp/zlib-1.2.7 \ 
   --with-openssl=/lamp/openssl-1.0.2
   ```

3. 增加nginx命令

   ```bash
   ln -s /usr/local/services/nginx-1.13.12/sbin/nginx /bin/nginx
   ```

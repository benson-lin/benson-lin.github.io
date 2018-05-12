---
title: "PHP 安装 和 PHP-FPM  启动"
layout: post
date: 2018-05-10
tags:
- Mysql
- PHP
categories: PHP
blog: true
excerpt: linux上php的安装
---

# PHP安装

1. 先创建安装的目录和下载安装包，并解压

   ```bash
   wget -O /tmp/php-7.2.5.tar.gz http://cn2.php.net/get/php-7.2.5.tar.gz/from/this/mirror
   mkdir /usr/local/services/php-7.2.5
   cd /tmp
   tar -zxvf php-7.2.5.tar.gz
   cd php-7.2.5
   ```

   

2. 使用./configure生成makefile文件，扩展可以增加需要删除或增加，如果提示确实依赖则按第3步安装

   ```bash
   ./configure --prefix=/usr/local/services/php-7.2.5 \
    --with-curl \
    --with-freetype-dir \
    --with-gd \
    --with-gettext \
    --with-iconv-dir \
    --with-kerberos \
    --with-libdir=lib64 \
    --with-libxml-dir \
    --with-mysqli \
    --with-openssl \
    --with-pcre-regex \
    --with-pdo-mysql \
    --with-pdo-sqlite \
    --with-pear \
    --with-png-dir \
    --with-xmlrpc \
    --with-xsl \
    --with-zlib \
    --enable-fpm \
    --enable-bcmath \
    --enable-libxml \
    --enable-inline-optimization \
    --enable-mbregex \
    --enable-mbstring \
    --enable-opcache \
    --enable-pcntl \
    --enable-shmop \
    --enable-soap \
    --enable-sockets \
    --enable-sysvsem \
    --enable-xml \
    --enable-zip \
    --enable-intl
   ```

   

3. 根据需要安装扩展依赖，先yum search xxx,在yum intall libxxx libxxx-devel 

   ```bash
   yum search xslt
   yum install libxslt libxslt-devel
   ```

   一般需要安装的如下：

   ```bash
   yum -y install gcc gcc-c++ autoconf automake libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel mysql pcre-devel openssl openssl-devel libxslt libxslt-devel libicu libicu-devel
   
   ```

4. 编译安装

   ```bash
   make &&  make install
   ```

5. 完成后就可以在/usr/local/services/php-7.2.5目录下看到安装的内容

6. 在/bin增加php命令，以使用php命令，增加后/usr/bin下也能看到软链接。（PS:如果需要显示全路径可以在/etc/profile中增加 `export PS1='[\u@\h \w]\$ '`）

   ```bash
   root@VM_31_175_centos php-7.2.5]# ln -s /usr/local/services/php-7.2.5/bin/php /bin/php
   root@VM_31_175_centos php-7.2.5]# ll /bin/php
   lrwxrwxrwx 1 root root 37 May  9 22:00 /bin/php -> /usr/local/services/php-7.2.5/bin/ph
   root@VM_31_175_centos php-7.2.5]# whereis php
   php: /usr/bin/php
   ```

7. 将php安装包的配置文件php.ini-production放到php安装目录下，并重命名为php.ini

   ```bash
   [root@VM_31_175_centos /usr/local/services/php-7.2.5]# php -i | grep php.ini
   Configuration File (php.ini) Path => /usr/local/services/php-7.2.5/lib
   [root@VM_31_175_centos /usr/local/services/php-7.2.5]# cp /tmp/php-7.2.5/php.ini-production /usr/local/services/php-7.2.5/lib/
   [root@VM_31_175_centos /usr/local/services/php-7.2.5]# cp /tmp/php-7.2.5/php.ini-development /usr/local/services/php-7.2.5/lib/
   [root@VM_31_175_centos /usr/local/services/php-7.2.5/lib]# php -v
   PHP 7.2.5 (cli) (built: May  9 2018 20:51:27) ( NTS )
   Copyright (c) 1997-2018 The PHP Group
   Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
   ```

   可以看到安装时的配置

   ```bash
   php -i |grep 'Configure'
   ```

   

   # 启动php-fpm

   1. 运行`/usr/local/services/php-7.2.5/sbin/php-fpm`报错 ，因为没有复制配置文件

      ```bash
      [root@VM_31_175_centos /usr/local/services/php-7.2.5/sbin]# cp /usr/local/services/php-7.2.5/etc/php-fpm.conf.default /usr/local/services/php-7.2.5/etc/php-fpm.conf
      ```

   2. 编辑php-fpm.conf，开启pid

      ```bash
      vim php-fpm.conf
      pid = run/php-fpm.pid
      ```

   3. 再次执行，报错

      ```bash
      [root@VM_31_175_centos /usr/local/services/php-7.2.5/sbin]# /usr/local/services/php-7.2.5/sbin/php-fpm
      [09-May-2018 22:44:03] WARNING: Nothing matches the include pattern '/usr/local/services/php-7.2.5/etc/php-fpm.d/*.conf' from /usr/local/services/php-7.2.5/etc/php-fpm.conf at line 125.
      [09-May-2018 22:44:03] ERROR: No pool defined. at least one pool section must be specified in config file
      [09-May-2018 22:44:03] ERROR: failed to post process the configuration
      [09-May-2018 22:44:03] ERROR: FPM initialization failed
      ```

   4. 进入目录`/usr/local/services/php-7.2.5/etc/php-fpm.d/`，发现还是配置文件问题，拷贝一份

      ```bash
      [root@VM_31_175_centos /usr/local/services/php-7.2.5/sbin]# ll /usr/local/services/php-7.2.5/etc/php-fpm.d/
      total 20
      -rw-r--r-- 1 root root 18914 May  9 21:57 www.conf.default 
      [root@VM_31_175_centos /usr/local/services/php-7.2.5/sbin]# cp /usr/local/services/php-7.2.5/etc/php-fpm.d/www.conf.default /usr/local/services/php-7.2.5/etc/php-fpm.d/www.conf 
      
      ```

   5. 再次执行`/usr/local/services/php-7.2.5/sbin/php-fpm`,查看进程，发现成功了

      ```bash
      [root@VM_31_175_centos /usr/local/services/php-7.2.5/sbin]# ps -ef |grep php-fpm
      root     18444     1  0 22:47 ?        00:00:00 php-fpm: master process (/usr/local/services/php-7.2.5/etc/php-fpm.conf)
      nobody   18445 18444  0 22:47 ?        00:00:00 php-fpm: pool www
      nobody   18446 18444  0 22:47 ?        00:00:00 php-fpm: pool www
      root     18451 26964  0 22:47 pts/0    00:00:00 grep --color=auto php-fpm
      ```

   6. www.conf可以配置启动的用户

      ```bash
      vim /usr/local/services/php-7.2.5/etc/php-fpm.d/www.conf 
      user = nobody
      group = nobody
      ```

   7. 重启php-fpm：(pid可以在`/usr/local/services/php-7.2.5/etc/php-fpm.conf`中查看到php-fpm.pid所在目录，其实就是ps中看到的master进程的id，即18444)

      ```bash
      kill -USR2 `cat /usr/local/services/php-7.2.5/var/run/php-fpm.pid`
      
      
      [global]
      ; Pid file
      ; Note: the default prefix is /usr/local/services/php-7.2.5/var
      ; Default Value: none
      pid = run/php-fpm.pid
      ```


   

   

   

   
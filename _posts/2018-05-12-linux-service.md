---
title: "Linux 创建服务"
layout: post
date: 2018-05-12
tag:
- Linux
categories: LINUX
blog: true
excerpt: 如何在linux上使用 service xxx start/stop/restart的命令呢？这篇文章就是说这个的
---

# Linux 创建service

以使用service xxx start/stop/restart



```
TERM,INT 快速关闭

QUIT 从容关闭

HUP 平滑重启，重新加载配置文件

USR1 重新打开日志文件，在切割日志时用途较大

USR2 平滑升级可执行程序

WINCH 从容关闭工作进程
```



下面以创建nginx和php-fpm为例，假设nginx安装在/usr/local/services/nginx-1.13.12/下（如果已经创建了nginx命令，可以不用指定路径），php安装在/usr/local/services/php-7.2.5下



## 1. 创建nginx服务

1. 创建nginx文件，放到/etc/init.d/目录下

   ```sh
   #!/bin/sh
   # service nginx start/stop/restart/test
   
   
   start(){
           /usr/local/services/nginx-1.13.12/sbin/nginx
   }
   stop(){
           /usr/local/services/nginx-1.13.12/sbin/nginx -s stop
   }
   restart(){
          /usr/local/services/nginx-1.13.12/sbin/nginx -s reload
          # kill -HUP master-id
          # 从nginx.conf找到pid所在目录，我配置的是var/run/nginx.pid
          # kill -HUP `cat /usr/local/services/nginx-1.13.12/var/run/nginx.pid`
          # kill -HUP `ps -ef | grep "nginx: master" | head -n 1 | awk '{print $2}'`
   }
   test(){
       /usr/local/services/nginx-1.13.12/sbin/nginx -t
   }
   
   case "$1" in
   start)
           start
           ;;
   stop)
           stop
           ;;
   restart)
           restart
           ;;
   test)
           test
           ;;
   *)
           echo "Usage: $0 {start|restart|stop}"
           exit 1
           ;;
   esac
   ```

   

2. 将所有者改为root,并赋予执行权限

   ```
   chown root:root /etc/init.d/nginx
   chmod 755 /etc/init.d/nginx
   
   [root@VM_31_175_centos /etc/init.d]# ll
   total 40
   -rw-r--r-- 1 root root 15131 Sep 12  2016 functions
   -rwxr-xr-x 1 root root  2989 Sep 12  2016 netconsole
   -rwxr-xr-x 1 root root  6643 Sep 12  2016 network
   -rwxr-xr-x 1 root root   546 May 12 13:35 nginx
   -rwxr-xr-x 1 root root   612 May 12 14:17 php-fpm
   -rw-r--r-- 1 root root  1160 May 26  2017 README
   ```

   

## 2. 创建php-fpm服务

同样的创建php-fpm文件，放到/etc/init.d/目录下，并赋予执行权限

```sh
#!/bin/sh
# service php-fpm start/stop/restart


start(){
       /usr/local/services/php-7.2.5/sbin/php-fpm
}
stop(){
       kill `cat /usr/local/services/php-7.2.5/var/run/php-fpm.pid`
}
restart(){
       # 可以查看 /usr/local/services/php-7.2.5/etc/php-fpm.conf
       # (不一定是这个路径下，指的是启动fpm的配置文件)
       kill -USR2 `cat /usr/local/services/php-7.2.5/var/run/php-fpm.pid`
}


case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
restart)
        restart
        ;;
*)
        echo "Usage: $0 {start|restart|stop}"
        exit 1
        ;;
esac
```


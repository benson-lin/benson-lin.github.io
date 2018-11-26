---
title: "手把手搭建第一个基于docker的nginx服务"
layout: post
date: 2018-11-17
tags:
- Nginx
- Docker
categories: DOCKER
blog: true
excerpt: 如何在docker中部署nginx服务并启动呢？
---

## 手把手搭建第一个基于docker的nginx服务

首先说下为什么我们不用 dockerhub上的nginx(https://hub.docker.com/_/nginx/)，是因为我们可能有这种需求：就是在一个服务器已经运行了，因为业务需要，需要把这个业务移到docker中(如打包后对整个系统服务进行售卖，方便部署和管理等)。假如这个时候使用的服务是nginx。以nginx为例，说明在本地其实整个nginx是已经安装好的了，这个时候虽然也可以从dockerhub下载nginx，但是明显是从本地打包成tar.gz后拷贝到docker镜像中然后启动更加的容易(php同理)，也不会有什么兼容性问题。下面将假设我们的机器上已经编译安装了nginx，从 本地通过curl http://localhost 可以看到 Welcome to nginx! 字样了。



下面是安装好后的文件结构目录

```
[root@VM_165_144_centos /usr/local/services/nginx-1.14.0]# ll
total 40K
drwxr-xr-x  2 root   root 4.0K 2018-11-17 21:56:00 logs/
drwxr-xr-x  2 root   root 4.0K 2018-09-17 22:43:07 conf/
drwxr-xr-x  2 root   root 4.0K 2018-08-29 23:56:42 sbin/
drwx------ 12 nobody root 4.0K 2018-06-03 10:09:02 fastcgi_temp/
drwx------  2 nobody root 4.0K 2018-05-13 16:08:57 client_body_temp/
drwx------  2 nobody root 4.0K 2018-05-13 16:08:57 proxy_temp/
drwx------  2 nobody root 4.0K 2018-05-13 16:08:57 scgi_temp/
drwx------  2 nobody root 4.0K 2018-05-13 16:08:57 uwsgi_temp/
drwxr-xr-x  3 root   root 4.0K 2018-05-13 16:08:22 var/
drwxr-xr-x  2 root   root 4.0K 2018-05-13 16:02:06 html/
```



首先我们将nginx打包`cd /usr/local/services/nginx-1.14.0 tar -zcvf nginx-1.14.0.tar.gz ..`

创建docker_test文件夹，并将nginx压缩包放入package中

```
[root@VM_165_144_centos /data/home/bensonlin/docker_test]# tree
.
|-- Dockerfile
`-- package
    `-- nginx-1.14.0.tar.gz

1 directory, 2 files
```

编辑Dockerfile

```
[root@VM_165_144_centos /data/home/bensonlin/docker_test]# vim Dockerfile 
FROM centos:centos7

ADD package/nginx-1.14.0.tar.gz /usr/local/services/nginx-1.14.0

EXPOSE 80
```

构建镜像，拿到镜像ID为1d55926ed281

```
[root@VM_165_144_centos /data/home/bensonlin/docker_test]#  docker build -t nginx .
Sending build context to Docker daemon 6.269 MB
Step 1/3 : FROM centos:centos7
 ---> 75835a67d134
Step 2/3 : ADD package/nginx-1.14.0.tar.gz /usr/local/services/nginx-1.14.0
 ---> ee4615067885
Removing intermediate container dbab572fcfb8
Step 3/3 : EXPOSE 80
 ---> Running in 8357c1b70faf
 ---> 1d55926ed281
Removing intermediate container 8357c1b70faf
Successfully built 1d55926ed281

[root@VM_165_144_centos /data/home/bensonlin/docker_test]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              1d55926ed281        25 seconds ago      218 MB

```

看下里面的目录结构先（通过创建一个命令行容器）`docker run -it nginx /bin/bash`



启动nginx，命令中的nginx为镜像名，`/usr/local/services/nginx-1.14.0/sbin/nginx`为容器内nginx命令位置

```
[root@VM_165_144_centos /data/home/bensonlin/docker_bensonlin]# docker run --name nginx_ct -d -p 80:80  nginx /usr/local/services/nginx-1.14.0/sbin/nginx  -g 'daemon off;'
6ea6c98fed2abf47dfa73cd3f098825813a2853504db37216d695fed373dd55f
```

查看正在运行的容器，可以看到容器ID是6ea6c98fed2a，容器名为nginx_ct

```
[root@VM_165_144_centos /data/home/bensonlin/docker_bensonlin]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
6ea6c98fed2a        nginx               "/usr/local/servic..."   8 seconds ago       Up 8 seconds        0.0.0.0:80->80/tcp   nginx_ct
```

此时可以通过`curl http://localhost`就可以看到Welcome to nginx! 字样

如果要关闭容器可以通过`docker stop nginx_ct` 或者通过指定CONTAINER ID 停止。还使用`docker start nginx_ct`启动和使用`docker restart nginx_ct`重启



问题来了，虽然我curl进入了目录，但是我要怎么进入容器看到里面的内容呢。这里就涉及如何进入容器的问题了，`docker exec -it nginx_ct /bin/bash`，只需这样就可以了，是不是很简单

可以看到nginx已经创建了，并且已经跑起来了

```
[root@VM_165_144_centos /data/home/bensonlin/docker_bensonlin]# docker exec -it nginx_ct /bin/bash
[root@6ea6c98fed2a /]# cd /usr/local/services/nginx-1.14.0/
[root@6ea6c98fed2a nginx-1.14.0]# pwd
/usr/local/services/nginx-1.14.0
[root@6ea6c98fed2a nginx-1.14.0]# ll 
total 40
drwx------  2 nobody root 4096 May 13  2018 client_body_temp
drwxr-xr-x  2 root   root 4096 Nov 17 14:05 conf
drwx------ 12 nobody root 4096 Jun  3 02:09 fastcgi_temp
drwxr-xr-x  2 root   root 4096 May 13  2018 html
drwxr-xr-x  1 root   root 4096 Nov 17 15:33 logs
drwx------  2 nobody root 4096 May 13  2018 proxy_temp
drwxr-xr-x  2 root   root 4096 Aug 29 15:56 sbin
drwx------  2 nobody root 4096 May 13  2018 scgi_temp
drwx------  2 nobody root 4096 May 13  2018 uwsgi_temp
drwxr-xr-x  1 root   root 4096 May 13  2018 var
[root@6ea6c98fed2a nginx-1.14.0]# ps -ef | grep nginx
root         1     0  0 15:38 ?        00:00:00 nginx: master process /usr/local/services/nginx-1.14.0/sbin/nginx -g daemon off;
nobody       5     1  0 15:38 ?        00:00:00 nginx: worker process
nobody       6     1  0 15:38 ?        00:00:00 nginx: worker process
nobody       7     1  0 15:38 ?        00:00:00 nginx: worker process
nobody       8     1  0 15:38 ?        00:00:00 nginx: worker process
nobody       9     1  0 15:38 ?        00:00:00 nginx: worker process
nobody      10     1  0 15:38 ?        00:00:00 nginx: worker process
nobody      11     1  0 15:38 ?        00:00:00 nginx: worker process
nobody      12     1  0 15:38 ?        00:00:00 nginx: worker process
nobody      13     1  0 15:38 ?        00:00:00 nginx: worker process
nobody      14     1  0 15:38 ?        00:00:00 nginx: worker process
root        28    15  0 15:39 ?        00:00:00 grep --color=auto nginx
```



这里有个注意点，如果我们不进入指定容器，其实可以在build之后就通过指定镜像名创建容器看到镜像内的内容，比如核对目录结构和相关权限等等.`docker -it nginx /bin/bash`，用法其实和启动nginx一致的，但是如果要进入已经创建的容器，则需要通过`docker exec`进入（不要用docker attach，这个是进入容器，exit之后会容器会终止），否则在不同容器是看不到对应的nginx服务的

如果需要删除容器，`docker rm nginx_ct `  ，如果容器正在运行。删除容器前需要先停止运行容器`docker stop nginx_ct`

```
Error response from daemon: You cannot remove a running container 6ea6c98fed2abf47dfa73cd3f098825813a2853504db37216d695fed373dd55f. Stop the container before attempting removal or use -f
```

删除镜像：`docker rmi nginx`
---
title: "Mysql安全"
layout: post
date: 2018-03-13
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: 安全相关内容，根据mysql官方文档翻译并归纳
---

# Mysql安全

### 一、安全指南通用方法

1. 不要给任何人(除了root账号外)访问`mysql.user`表
2. 学习mysql访问权限是如何工作的，使用grant和revoke控制权限，不要授予非必要之外的其他权限，不要授权权限给所有的主机host

   - 尝试`mysql -u root`能不能直接登录，如果没有应该设置密码

   - 使用`show grants`查看其他用户拥有哪些权限，使用`revoke`移除非必要的权限
3. 不要在数据库中使用明文，应当使用hash函数对密码进行计算，同时为了防止被字典强破，可以加上盐进行计算保存(hash(hash(password)+salt)
4. 不要使用纯英文单词作为密码。可以选择一个句子中的每个单词的第一个单词作为密码
5. 将数据库放到防火墙之下，可以减少至少50%的漏洞利用。
   - 试着从网络中使用如`	nmap`的工具扫描端口(如`nmap 127.0.0.1`)，mysql默认使用3306端口，这个端口不应该被不信任的主机访问。一个简单的检查方法是，尝试从远程机器使用`telnet server_host 3306`命令来测试，其中server_host为mysql所在的主机名或IP地址
   - 如果telnet挂起或连接被拒绝，说明端口阻塞了，正是符合预期的。如果获得连接和收到了包，说明端口开放了，应该从防火墙和路由器上进行关闭。除非有好的理由开放这个端口。
6. 不要信任任何由用户输入的进入mysql的数据，应当使用适当的编程防御措施。
7. 不要在网络上传输未加密的数据，使用诸如SSL或SSH的加密协议。mysql支持内部的ssl连接，另一种方法是使用SSH端口转发来创建一个加密的通信(压缩)隧道。
8. 学习使用tcpdump和strings工具，在大多数情况下,您可以检查MySQL数据流是否加密通过发出一个命令如下。`tcpdump -l -i eth0 -w - src or dst port 3306 | strings`



```shell
[root@VM_172_210_centos ~]# nmap 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2018-03-08 18:59 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (-1900s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds

```

从外部扫描

```shell
C:\Users\bensonlin>nmap 118.24.73.69

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-08 19:11 ?D1ú±ê×?ê±??
Nmap scan report for 118.24.73.69
Host is up (0.041s latency).
Not shown: 988 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
42/tcp   filtered nameserver
135/tcp  filtered msrpc
139/tcp  filtered netbios-ssn
445/tcp  filtered microsoft-ds
593/tcp  filtered http-rpc-epmap
1025/tcp filtered NFS-or-IIS
1068/tcp filtered instl_bootc
3128/tcp filtered squid-http
3306/tcp open     mysql
4444/tcp filtered krb524
5800/tcp filtered vnc-http

C:\Users\bensonlin>telnet 118.24.73.69 3306
R
 5.5.56-MariaDB^UF<;RjAD??p(r)s*s)oty&mysql_native_password
```

## 二、保证密码安全

###  1、终端用户密码安全指南

1. 使用 `mysql_config_editor`安全登录工具，会生成加密文件

2. ` mysql -u francis -p db_name`，尽量不要在命令行使用密码，而是通过提示输入密码的形式，否则密码可能防止在`.bash_history`文件中被查看

3. 保存密码在一个my.cnf，并设置600的权限（注意读取的顺序）。MySQL忽略人人可写的配置文件。这是故意的，是一个安全措施。

```shell
# The following options will be passed to all MySQL clients  
[client]  
host=localhost                                                                  
password=1111 
user=root   
port=3306  


chmod 600 .my.cnf

shell> mysql --defaults-file=/etc/my.cnf

[root@VM_172_210_centos ~]# ps -ef | grep mysql
mysql    16820     1  0 19:38 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql    17104 16820  0 19:38 ?        00:00:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock

# 启动时没指定my.cnf路径时(mysqld中没有--defaults-file=选项),
# 默认my.cnf读取顺序(mysql --help | grep 'Default options' -A 1)
File Name	       Purpose
/etc/my.cnf	        Global options
/etc/mysql/my.cnf	Global options
SYSCONFDIR/my.cnf	Global options
$MYSQL_HOME/my.cnf	Server-specific options (server only)
defaults-extra-file	The file specified with --defaults-extra-file, if any
~/.my.cnf	        User-specific options
~/.mylogin.cnf	    User-specific login path options (clients only)

```

   

4. mysql客户端会保存语句的历史到`.mysql_history`中，保存在home目录下，为了能够保证安全，应该也对这个文件设置600权限或400权限

5. 不建议：保存密码到``MYSQL_PWD` `环境变量(mysql命令的默认密码)中，这个是及其不安全，有些版本的ps命令能获取正在运行的进程的环境(如将mysql命令&挂起，ps –f)

### 2、管理员密码安全指南

1. `mysql.user`保存着用户的密码，这张表不应该被授予非管理员账户
2. 用户的密码应该定时过期，强制进行重置
3. 使用``validate_password` 插件强制接收一个可接受的密码，避免弱密码
4. 有权限修改插件目录（系统变量plugin_dir）或者修改my.cnf文件(能指定插件目录位置)的用户，可以替换插件和修改由插件提供的功能，包括身份认证插件
5. 文件(如日志文件)这种可能写入过密码的需要被保护


### 3、密码和日志

使用了`CREATE USER`, `GRANT` `SET PASSWORD`语句或者 `PASSWORD()`函数会被Mysql服务器记录下来。

```
CREATE USER ... IDENTIFIED BY ...
ALTER USER ... IDENTIFIED BY ...
GRANT ... IDENTIFIED BY ...
SET PASSWORD ...
SLAVE START ... PASSWORD = ...
CREATE SERVER ... OPTIONS(... PASSWORD ...)
ALTER SERVER ... OPTIONS(... PASSWORD ...)
```

为了进行通用的查询语句的记录，可以使用`--log-raw`参数为false(默认)禁止密码写入。如果开启的话对诊断从客户端接收到的语句很有用，但是不建议在生产环境使用。

基于安全原因，日志文件应保证只有合法用户能够查看。



##  三、使MySQL免受攻击者的攻击

1. 在mysql客户端和服务端之间使用SSL(`<http://www.openssh.org/`)

2. 为了mysql的安全，强烈推荐使用下面建议：

   - 确保所有的mysql用户有密码，否则用户可以在客户端指定任意用户名登录
   - 确保唯一对数据库目录具有读或写权限的Unix账号是用于运行mysqld的账号。
   - 不要用root用户启动mysql服务器，因为这样会使任何拥有`File`权限的用户能够在服务器上用root用户身份创建文件，如/root/.bashrc。可使用的命令包括`load data infile,select ... into outfile,load file()`函数.下面这个是没权限：

   ```
   MariaDB [test_mgr]> load data infile '/root/1.txt' into table test1;
   ERROR 13 (HY000): Can't get stat of '/root/1.txt' (Errcode: 13)
   ```

   - mysqld应该作为一个普通的、非特权用户来运行。你可以创建一个单独的Unix账号，名为mysql，该账号专门用于管理MySQL，从而使一切更加安全。要以不同的Unix用户启动mysqld，请在my.cnf选项文件的组[mysqld]中添加一个用户选项，例如：

     ```
     [mysqld]
     user=mysql
     ```

     无论你是手动还是使用mysqld_safe或mysql.server启动服务器，该选项都将使得服务器以指定的用户身份启动

   - 不要把FILE权限给非管理员账户。具有此权限的任何用户都可以以mysqld进程的有效用户ID的身份在文件系统的任何位置写入文件。这包括服务器的数据目录，其中含有实现权限表的文件。为了使「file」权限操作更安全，select ... into outfile生成的文件不会覆盖现有的文件。「file」权限也可以用来读取全局可读或者是MySQL服务器（mysqld进程的有效用户ID）具有读权限的所有文件。使用此权限，你可以将任何文件读入数据库表。这可能会被滥用，例如，使用「load data」将/etc/passwd加载到表中，然后使用select语句进行显示。为了限制可以读取和写入文件的位置，请将系统变量「secure_file_priv」设置为特定的目录。

   - 不要把`process`或`super`权限给非管理员用户。mysqladmin的processlist和`show processlist`会展示正在执行的语句，其中可能包含密码的操作。mysqld 为SUPER权限的用户提供一个额外的链接，因此root用户能够登录监控服务器的行为，即使所有的连接都被使用，SUPER权限能够终止客户连接改变系统变量和控制服务器复制

   - 不允许使用到库表的符号链接（可以用 `--skip-symbolic-links禁用`），当使用root允许mysqld时非常重要，会导致任何对服务器数据目录具有写权限的人都可以删除系统中的任何文件(MYISAM)

   - 存储过程和视图应该按照特定安全指令书写(https://dev.mysql.com/doc/refman/5.7/en/stored-programs-security.html)

   - 如果不相信你的DNS，grant时应该使用IP地址，而不是主机名。在任何情况下，应该非常小心的使用对使用通配符的主机授权

   - 如果想限制单个账户的连接数，能够在mysqld中设置`max_user_connections`变量。为了限制一个用户的使用范围，CREATE USER和ALTER USER语句也支持资源控制。

   - 如果插件目录对于服务器来说是可写的，那么用户可以使用select ... into dumpfile将可执行代码写入目录中的文件。为了防止这种情况，你可以设置「plugin_dir」目录仅对服务器开放读权限，或者将「--secure-file-priv」设置为可以安全地进行select写入的目录。




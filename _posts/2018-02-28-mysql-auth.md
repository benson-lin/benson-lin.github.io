---
title: "Mysql权限控制"
layout: post
date: 2018-02-28
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: grant, revoke, create user等
---

# Mysql权限控制

1. 创建用户：`create user bensonlin@localhost identified by 'bensonlin'`

2. 删除用户：`drop user bensonlin@localhost ` 

3. 修改用户：` rename user bensonlin@localhost to zhangsan@localhost`

4. 修改用户密码：` set password for bensonlin@localhost=password('newpwd');`

5. 展示某个用户的权限：`show grants for bensonlin@localhost`，查看自己：`show grants`

   新创建的用户无任何权限，只能连上库：

   `| GRANT USAGE ON *.* TO 'bensonlin'@'localhost' IDENTIFIED BY PASSWORD '*F99EFF78DB917AFD7F2D037EC4CC7041406B33D5' |`

6. 权限分配：

   1. 只授予某个表的查询某些字段：`grant select(id, name, age) on mysql_test.user to bensonlin@localhost`
   2. 授予某个表查询和修改：`grant select,update on mysql_test.user to bensonlin@localhost`
   3. 授予所有表的所有权限：`grant all on mysql_test.* to bensonlin@localhost`
   4. 授予创建用户权限：`grant create user on *.* to bensonlin@localhost`

7. 权限转移：`grant select,update on mysql_test.user to bensonlin@localhost with grant option;`

8. 权限撤销：`revoke select,update on mysql_test.user from bensonlin@localhost `


---
title: "Mysql备忘"
layout: post
date: 2017-02-06
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: 包含 mysqldump 操作等，持续更新
---

# Mysql备忘



## 一、Mysql dump 操作

（1）导出整个数据库(包括数据库中的数据）

    mysqldump -u username -p dbname > dbname.sql    

（2）导出数据库结构（不含数据）

    mysqldump -u username -p -d dbname > dbname.sql    

（3）导出数据库中的某张数据表（包含数据）

    mysqldump -u username -p dbname tablename > tablename.sql    

（4）导出数据库中的某张数据表的表结构（不含数据）

    mysqldump -u username -p -d dbname tablename > tablename.sql  

## 二、 case when语句


两种方式：

```
CASE sex
         WHEN '1' THEN '男'
         WHEN '2' THEN '女'
ELSE '其他' END
（列名是sex，如果放在case后，则显示在屏幕上的列名即为列名sex）


CASE WHEN sex = '1' THEN '男'
         WHEN sex = '2' THEN '女'
ELSE '其他' END
 （列名是sex，如果放在when后，若是不为整个CASE WHEN语句写个别名的话，则显示在屏幕上的列名即为整个CASE WHEN语句，因此我们经常会写as作为列名，如下面的实例）

```




下面是实例, 由于有一列type标识支出收入（type=1表示收入，type2表示支出），money列都用正数去表示；问题来了，如果要统计余额，要怎么做，也就是说支出类型要转为负数，因此case when就派上用场了。

```sql
select ca.name, sum(case when type=1 then money else (-1)*money end) as money
	from t301_income_expend_record re join t201_account_category ca on re.account_category_id =  ca.id 
	where user_id=1 group by account_category_id
```
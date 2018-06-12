---
title: "Mysql语句备忘"
layout: post
date: 2018-03-30
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: mysql常见sql语句合集
---

# Mysql备忘


## dump 操作

（1）导出整个数据库(包括数据库中的数据）

    mysqldump -u username -p dbname > dbname.sql    

（2）导出数据库结构（不含数据）

    mysqldump -u username -p -d dbname > dbname.sql    

（3）导出数据库中的某张数据表（包含数据）

    mysqldump -u username -p dbname tablename > tablename.sql    
    mysqldump -uroot -proot --databases db1 --tables a1 a2  >/tmp/db1.sql

（4）导出数据库中的某张数据表的表结构（不含数据）

    mysqldump -u username -p -d dbname tablename > tablename.sql  
    mysqldump -uroot -proot --no-data --databases db1 >/tmp/db1.sql

（5）完整语句

    mysqldump -u<username> -p<passwrod> -h<host> -P<port> <dbname> <tablename> > /tmp/test.sql

 ### 对于导入慢的问题

使用`mysqldump -uroot -p'123' --all-database >all.sql `导出所有数据库，在目录数据库中执行

```
mysql> show variables like 'max_allowed_packet';
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 4194304 |
+--------------------+---------+
1 row in set, 1 warning (0.00 sec)

mysql> show variables like 'net_buffer_length';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| net_buffer_length | 16384 |
+-------------------+-------+
1 row in set, 1 warning (0.00 sec)
```

记录下参数（max_allowed_packet和net_buffer_length不能比目标数据库的设定数值大，否则可能出错）

`mysqldump -uroot -p'123' db table -w "time>'xxxx'" --default-character-set=utf8 --max_allowed_packet=1047552 --net_buffer_length=16384 >all.sql`，在导出的时候使用--net_buffer_length 和--max_allowed_packet  ，大于服务端允许最大值时可能报错。



包消息缓冲区初始化为net_buffer_length字节，但需要时可以增长到max_allowed_packet字节

--net_buffer_length ：  TCP/IP和socket连接的缓存大小

--max_allowed_packet ：服务器发送和接受的最大包长度。一个packet里面可能包含多个row data,但是至少应该存下一个row data（insert语句默认为一次性多条insert，大于最大值才拆分成多个insert，但如果单条数据大小大于最大值将报错）

https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_net_buffer_length

也就是此参数指定了一个缓存区的大小，用来存放用户发送的SQL语句。若接收的SQL语句大于这个缓存区，则自动增加大小，直到max_allowed_packet



**修改`max_allowed_packet`**

```
[mysqldump] 
max_allowed_packet = 16M 
```





### 查看表占用的大小

```mysql
SELECT 
    table_name AS `Table`, 
    round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
FROM information_schema.TABLES 
WHERE table_schema = "dbname"
    AND table_name = "tablename";
```

结果如下：

```
+-----------------------------------+------------+
| Table                             | Size in MB |
+-----------------------------------+------------+
| tablename                         |     367.08 |
+-----------------------------------+------------+
1 row in set (0.00 sec)
```



查看每个数据库每个表的大小，按大小降序

```mysql
SELECT 
     table_schema as `Database`, 
     table_name AS `Table`, 
     round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
FROM information_schema.TABLES 
ORDER BY (data_length + index_length) DESC;
```



如果只是看某个库的每个表大小

```mysql
SELECT 
     table_schema as `Database`, 
     table_name AS `Table`, 
     round(((data_length + index_length) / 1024 / 1024), 2) `Size in MB` 
FROM information_schema.TABLES  WHERE table_schema = "dbname"
ORDER BY (data_length + index_length) DESC;	
```





## mysql执行并快速导出

````mysql
mysql -h *** -u *** -P 3314 -p'***' --default-character-set=utf8 *** < input.txt > output.txt
````





##  select并导出到csv文件

```sql
select * from test where orderId='3015' INTO OUTFILE 'C:/ProgramData/MySQL/MySQL Server 5.7/Uploads/test.csv' FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n';
```

如果报错可以查看 secure_file_priv 变量，文件的OUTFILE路径需要放置到该目录下：

```sql
show variables like '%secure%';
+--------------------------+------------------------------------------------+
| Variable_name            | Value                                          |
+--------------------------+------------------------------------------------+
| require_secure_transport | OFF                                            |
| secure_auth              | ON                                             |
| secure_file_priv         | C:\ProgramData\MySQL\MySQL Server 5.7\Uploads\ |
+--------------------------+------------------------------------------------+
3 rows in set
```



## 导入文件到mysql

1. load data：可以导入普通文本
2. `source d:/a.sql`或`\. d:/a.sql`：导入sql文件,记得不能加引号在路径两边

`load data local infile 'D:/111111111111important/11.txt' into table test1;`

文件的格式要符合表格格式，每个字段直接用tab分隔；

如test1表是这样的：

```sql
mysql> desc test1;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(12) | YES  | MUL | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
```

导入的txt是这样的：

```txt
1	test1
2	test2
```

结果如下：

```sql
mysql> select * from test1;
+----+--------+
| id | name   |
+----+--------+
 | 1 | test1
|  2 | test2  |
+----+--------+
2 rows in set (0.00 sec)
```

如果要指定特定的标识符作为行的分隔符(如\r\n)或指定字符分隔字段，可以这样用：

```sql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
    -> LINES TERMINATED BY '\r\n';
mysql> LOAD DATA INFILE 'data.txt' INTO TABLE table2
    -> FIELDS TERMINATED BY ',';
```

如果要指定编码或插入字符的顺序：

```sql
mysql> load data local infile 'D:/111111111111important/11.txt' into table test1
 character set utf8 fields terminated by ',' (name,id) ;
```





## 查看最慢sql

查看任务数量：

```sql
select count(1) from information_schema.processlist;
```

查看非sleep最多的sql：

```sql
select info, count(1) as count from information_schema.processlist  where command <>'Sleep' group by info order by count desc limit 3;
```

查看非sleep占用时间最长的sql:

```sql
select * from information_schema.processlist where command <>'Sleep' order by time DESC limit 15;
```

查看执行时间最长的在跑的sql，如果sql执行时间长，并且有并发量（有多条同样的sql在同时执行），基本可以确认是这条

如果需要kill掉，可以先执行下面语句获取ID，然后在notepad中批量替换为`kill id;`

```sql
select ID from information_schema.processlist where COMMAND<>'Sleep' 
	order by Time desc limit 15;
+-----------+
| ID        |
+-----------+
| 280334095 | 
| 280334097 | 
| 280338346 | 
| 280340359 | 
| 280340254 | 
| 280340257 | 
| 280337349 | 
```

或者

```sql
select concat('Kill ', id, ';') from information_schema.processlist where command <>'Sleep' order by time DESC limit 15;
```



## 索引相关操作

```sql
-- 1. 添加主键索引
ALTER TABLE `table_name` ADD PRIMARY KEY (`column`) 

-- 2. 唯一索引
ALTER TABLE `table_name` ADD UNIQUE (`column`) 

-- 3. 普通索引
ALTER TABLE `table_name` ADD INDEX index_name (`column`) 

-- 4. 多列索引
ALTER TABLE `table_name` ADD INDEX index_name (`column1`, `column2`, `column3`)
-- 比如 因为一般情况下名字的长度不会超过 10，这样会加速索引查询速度，还会减少索引文件的大小，提高 INSERT 的更新速度。
-- 相当于建立了 (vc_Name,vc_City,i_Age) (vc_Name,vc_City) (vc_Name) 三个组合索引（最左前缀原则）。
ALTER TABLE myIndex ADD INDEX name_city_age (vc_Name(10),vc_City,i_Age);

-- 5. 全文索引
ALTER TABLE `table_name` ADD FULLTEXT (`column`) 
```





## case when语句

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

## 

## 获取表信息

`show table status like 'table_name' \G;`

```
mysql> show table status like 't100_order_config' \G;
*************************** 1. row ***************************
           Name: t100_order_config
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 1
 Avg_row_length: 16384
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2017-09-13 16:51:36
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 派单配置
1 row in set (0.00 sec)
```

1. Name: 表名称
2. Engine: 表的存储引擎
3. Version: 版本
4. Row_format: 行格式。对于MyISAM引擎，这可能是Dynamic，Fixed或Compressed。动态行的行长度可变，例如Varchar或Blob类型字段。固定行是指行长度不变，例如Char和Integer类型字段。
5. Rows: 表中的行数。对于非事务性表，这个值是精确的，对于事务性引擎，这个值通常是估算的。
6. Avg_row_length: 平均每行包括的字节数
7. Data_length: 整个表的数据量(单位：字节)
8. Max_data_length: 表可以容纳的最大数据量
9. Index_length: 索引占用磁盘的空间大小
10. Data_free: 对于MyISAM引擎，标识已分配，但现在未使用的空间，并且包含了已被删除行的空间。
11. Auto_increment: 下一个Auto_increment的值
12. Create_time: 表的创建时间
13. Update_time: 表的最近更新时间
14. Check_time: 使用 check table 或myisamchk工具检查表的最近时间
15. Collation: 表的默认字符集和字符排序规则
16. Checksum: 如果启用，则对整个表的内容计算时的校验和
17. Create_options: 指表创建时的其他所有选项
18. Comment: 包含了其他额外信息，对于MyISAM引擎，包含了注释，如果表使用的是innodb引擎,将显示表的剩余空间。如果是一个视图，注释里面包含了VIEW字样。



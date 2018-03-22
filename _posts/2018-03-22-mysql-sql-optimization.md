---
title: "Mysql语句优化"
layout: post
date: 2018-03-22
tags:
- Mysql
categories: MYSQL
blog: true
excerpt: mysql常用优化合集
---

# Mysql语句优化

### 1. in导致速度太慢问题: 改用join临时表

语句的场景是这样的：需要统计前一天进行的某个操作（如提交初审）的总个数，且需要区分进行操作的是第一次还是非第一次（首次提交初审的次数/非首次提交初审的次数）；所以在查询的时候需要先查出前一天进行这个操作的行，然后在从全表获取每一行记录中进行这个操作的顺序。



原来的思路是：

先从t102_beian_operation表中找出某一天时间范围内(`operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59'`)且操作的标识是101（`operation = 101`）的订单号，因为订单号可以重复（一个订单号一天可能有多个同样的操作），所以进行的distinct操作。接着再找出这些订单号在全表中操作是101的所有行。



结果：

原语句使用in操作执行时间10-20s，慢的原因是因为总表个数为200多万条数据，即使加上operation = 101也还有29多万条记录。而in后面的子查询的结果为1300多条，这1300多条会每次都和29万条记录进行icp_order_id的比对，导致时间太慢

```sql


mysql> select count(*) from t102_beian_operation where operation = 101 and icp_order_id in ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101);
+----------+
| count(*) |
+----------+
|     2649 |
+----------+
1 row in set (19.39 sec)


mysql> select count(*) from t102_beian_operation;
+----------+
| count(*) |
+----------+
|  2616651 |
+----------+
1 row in set (0.43 sec)


mysql> select count(*) from t102_beian_operation where operation = 101 ;
+----------+
| count(*) |
+----------+
|   291329 |
+----------+
1 row in set (0.16 sec)


mysql>  select count(distinct(icp_order_id)) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101;
+-------------------------------+
| count(distinct(icp_order_id)) |
+-------------------------------+
|                          1349 |
+-------------------------------+
1 row in set (0.03 sec)


```

经过explain后可以发现扫描的行数特别的大。

```sql


mysql> explain select count(*) from t102_beian_operation where operation = 101;
+----+-------------+----------------------+------+-----------------+-----------------+---------+-------+--------+-------------+
| id | select_type | table                | type | possible_keys   | key             | key_len | ref   | rows   | Extra       |
+----+-------------+----------------------+------+-----------------+-----------------+---------+-------+--------+-------------+
|  1 | SIMPLE      | t102_beian_operation | ref  | index_operation | index_operation | 4       | const | 599112 | Using index |
+----+-------------+----------------------+------+-----------------+-----------------+---------+-------+--------+-------------+
1 row in set (0.00 sec)



mysql> explain select count(*) from t102_beian_operation where operation = 101 and icp_order_id in ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101);
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
| id | select_type        | table                | type           | possible_keys                                         | key                | key_len | ref   | rows   | Extra       |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
|  1 | PRIMARY            | t102_beian_operation | ref            | index_operation                                       | index_operation    | 4       | const | 598986 | Using where |
|  2 | DEPENDENT SUBQUERY | t102_beian_operation | index_subquery | index_icp_order_id,index_operate_time,index_operation | index_icp_order_id | 194     | func  |      8 | Using where |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
2 rows in set (0.00 sec)


mysql> explain select count(*) from t102_beian_operation where operation = 101 and icp_order_id in ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101);
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
| id | select_type        | table                | type           | possible_keys                                         | key                | key_len | ref   | rows   | Extra       |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
|  1 | PRIMARY            | t102_beian_operation | ref            | index_operation                                       | index_operation    | 4       | const | 592518 | Using where |
|  2 | DEPENDENT SUBQUERY | t102_beian_operation | index_subquery | index_icp_order_id,index_operate_time,index_operation | index_icp_order_id | 194     | func  |      8 | Using where |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
2 rows in set (0.00 sec)

```

解决方法，**使用join临时表的形式**，结果在1s内完成：

```
mysql> select count(*) from t102_beian_operation as a join ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101) as b on a.icp_order_id = b.icp_order_id   where operation = 101;
+----------+
| count(*) |
+----------+
|     2649 |
+----------+
1 row in set (0.17 sec)

```

原因可以从explain中看出，实际上是先获取临时表的所有订单号，然后在和已经筛选出来的全表operation=101的数据进行join，和使用in的形式差别很大：

```sql

mysql> explain select count(*) from t102_beian_operation as a join ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101) as b on a.icp_order_id = b.icp_order_id   where operation = 101;
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+----------------+-------+------------------------------+
| id | select_type | table                | type  | possible_keys                      | key                | key_len | ref            | rows  | Extra                        |
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+----------------+-------+------------------------------+
|  1 | PRIMARY     | <derived2>           | ALL   | NULL                               | NULL               | NULL    | NULL           |  1349 |                              |
|  1 | PRIMARY     | a                    | ref   | index_icp_order_id,index_operation | index_icp_order_id | 194     | b.icp_order_id |     8 | Using where                  |
|  2 | DERIVED     | t102_beian_operation | range | index_operate_time,index_operation | index_operate_time | 4       | NULL           | 35068 | Using where; Using temporary |
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+----------------+-------+------------------------------+
3 rows in set (0.03 sec)

```




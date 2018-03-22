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

### 1. in导致速度太慢问题

原语句使用in操作执行时间10-20s：

```


mysql> select count(*) from t102_beian_operation where operation = 101 and icp_order_id in ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101);
+----------+
| count(*) |
+----------+
|     2623 |
+----------+
1 row in set (11.76 sec)



mysql> explain select count(*) from t102_beian_operation where operation = 101 and icp_order_id in ( select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101);
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
| id | select_type        | table                | type           | possible_keys                                         | key                | key_len | ref   | rows   | Extra       |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
|  1 | PRIMARY            | t102_beian_operation | ref            | index_operation                                       | index_operation    | 4       | const | 592518 | Using where |
|  2 | DEPENDENT SUBQUERY | t102_beian_operation | index_subquery | index_icp_order_id,index_operate_time,index_operation | index_icp_order_id | 194     | func  |      8 | Using where |
+----+--------------------+----------------------+----------------+-------------------------------------------------------+--------------------+---------+-------+--------+-------------+
2 rows in set (0.00 sec)

```





```

mysql> explain select distinct(icp_order_id) from t102_beian_operation where operate_time >='2018-03-21 00:00:00' and operate_time <= '2018-03-21 23:59:59' and operation=101;
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+------+-------+------------------------------+
| id | select_type | table                | type  | possible_keys                      | key                | key_len | ref  | rows  | Extra                        |
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+------+-------+------------------------------+
|  1 | SIMPLE      | t102_beian_operation | range | index_operate_time,index_operation | index_operate_time | 4       | NULL | 35068 | Using where; Using temporary |
+----+-------------+----------------------+-------+------------------------------------+--------------------+---------+------+-------+------------------------------+
1 row in set (0.00 sec)


```






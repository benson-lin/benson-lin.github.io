---
title: "Redis Data types"
layout: post
date: 2016-03-29
permalink : /blogs/redis-data-types
tag:
- Redis
- Java
blog: true
---

## Data types Introduction

Redis并不是简单的key-value存储，实际上他是一个**数据结构服务器**，支持不同类型的值。其中包括 Strings, Lists, Sets, Hashes，Sorted sets。
可以翻译成 字符串，列表，集合，哈希，有序集合。
还有bitmaps和HyperLogLogs等。

这篇文章根据官方文档对数据类型的描述进行总结和一些细节的补充。

更多数据类型详细介绍也在[英文文档](http://redis.io/topics/data-types-intro)中。更多命令用法可以[点击这里](http://www.redis.io/commands)。也可以查看[中文文档](http://www.redis.cn/topics/data-types.html)，不过这个貌似是只是某个人注册了这个域名，翻译的不好，不像是官方的网站，而且目前没有翻译完整，所以还是建议大家阅读英文文档。

## Redis keys

Redis key值是二进制安全的，任何二进制序列都作为key值。空字符串也是有效key值。

下面是官方文档建议的key的规则

>
关于key的几条规则：
>
- 太长的键值不是个好主意，例如1024字节的键值就不是个好主意，不仅因为消耗内存，而且在数据中查找这类键值的计算成本很高。
- 太短的键值通常也不是好主意，如果你要用`u:1000:pwd`来代替`user:1000:password`，这没有什么问题，但后者更易阅读，并且由此增加的空间消耗相对于`key object`和`value object`本身来说很小。当然，没人阻止您一定要用更短的键值节省一丁点儿空间。
- 最好坚持一种模式。例如：`object-type:id:field`就是个不错的注意，像这样`user:1000:password`。我喜欢对多单词的字段名中加上一个点，就像这样：`comment:1234:reply.to`。

我补充一句，key 只要能够符合当前场景使用就可以了，这要求我们要考虑清楚对这个 key 的使用是否方便，所以下面的规则才规定我们不要用太短以表达清楚 key 的含义。而坚持一种模式无非就是让我们在根据 key 获取值时不会产生混淆，知道每个:隔开的指的是什么。太长影响效率就不用解释了。


## Strings

Redis的String类型是最简单的数据类型。

常见的操作有`SET`, `GET`, `MSET`, `MGET`命令。
其中`SET`和 `GET` 是对单个key的操作，而`MSET`和`MGET`是对多个key。

(m表示multiple，翻译成多个的，方便记忆，后面的命令也是按照这种命名规则)

```
127.0.0.1:6379> set str helloworld
OK
127.0.0.1:6379> get str
"helloworld"
127.0.0.1:6379> mset a 10 b 20 c 30
OK
127.0.0.1:6379> mget a c
1) "10"
2) "30"
127.0.0.1:6379> mget b
1) "20"
```

同时，还可以对数字型的字符串进行加减操作
如`INCR`, `INCRBY`, `DECR`, `DECRBY`

`INCR`操作是原子操作，就是说即使多个客户端对同一个key发出`INCR`命令，也决不会导致竞争的情况。开发过项目的人应该知道原子性有多重要吧，特别在多并发的情况下。最常见的情况，我们可以使用这个变量来实现访客的数量。


```
127.0.0.1:6379> incr b 
(integer) 21
127.0.0.1:6379> incr b 
(integer) 22
127.0.0.1:6379> incrby b 5
(integer) 27
127.0.0.1:6379> decr b
(integer) 26
```

## Redis expires: keys with limited time to live

可以指定key的过期时间，有两种方式：

一种是赋值后再设置过期时间 `expire key second`;
一种是在赋值的同时设定过期时间 `set key 100 ex 10`;

通过 `ttl key`查看剩余时间，单位为秒。过期后 key 将被删除，获取得到  nil


```
127.0.0.1:6379> set key value
OK
127.0.0.1:6379> expire key 30
(integer) 1
127.0.0.1:6379> ttl key
(integer) 27
127.0.0.1:6379> ttl key
(integer) 24
127.0.0.1:6379> get key
"value"
127.0.0.1:6379> ttl key
(integer) -2
127.0.0.1:6379> get key
(nil)

127.0.0.1:6379> set key 100 ex 10
OK
127.0.0.1:6379> get key
"100"
127.0.0.1:6379> ttl key
(integer) 5
```

## Lists

首先需要注意的是，这里的List是链表形式的，而非数组形式的。官网上明确指明，是`LinkedList`，而非`ArrayList`。链表形式的List可以让我们在常量时间(constant time, 应理解为O(1))内插入到列表的开头或者结尾。一个元素(element)加入到10个元素的列表和插入10万个元素的列表的速度是一样的。

或许我们会说，数组形式的List可以在`constant time`时间内获得某个index下标的值，而链表形式的List做不到这点，但是Redis为什么要采取这种方式呢？

因为数据库系统对在很短时间内将元素添加到一个很长的`List`中是至关重要的(crucial)，如果需要我们扩充数组，后果是不可想象的;文档中还有下面这个好处，但是英文水平有限，翻译不出来。

> Another strong advantage, as you'll see in a moment, is that Redis Lists can be taken at constant length in constant time.

如果需要迅速的定位到一个集合的中部位置，有另一个数据结构，即`Sorted sets`。

可以通过`LPUSH`和`RPUSH`命令分别在链表头和链表尾插入，对应的有`LPOP`和`RPOP`命令。可以通过`LRANGE`获取列表中的某个范围内的元素。

其中`lrange list 0 -1`表示列出全部的元素；如果列表中没有元素，将返回`nil`

```
127.0.0.1:6379> lpush list aaa
(integer) 1
127.0.0.1:6379> lpush list bbb
(integer) 2
127.0.0.1:6379> rpush list ccc
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "bbb"
2) "aaa"
3) "ccc"
127.0.0.1:6379> lrange list 1 2
1) "aaa"
2) "ccc"
//...pop操作
127.0.0.1:6379> lpop list
"ccc"
127.0.0.1:6379> lpop list
(nil)
```

当然，`*PUSH`命令能够同时插入多个值

```
127.0.0.1:6379> rpush list first 1 2 3 foo
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "first"
2) "1"
3) "2"
4) "3"
5) "foo"
```

Redis的列表可以做很多有趣的事：

1. 记录社交网站最新发布的Post，只需要用 `LPUSH`，同时使用`LTRIM`去创建一个永远不会超过指定元素数目的列表并同时记住最后的N个元素
2. 可以通过阻塞的PUSH和POP实现生产者-消费者模型（BRPOP和BLPOP），请自行翻阅官方文档

`LTRIM`和`RTRIM`命令用于获取一定范围内的元素，剔除范围外的元素。

```
127.0.0.1:6379> ltrim list 1 3
OK
127.0.0.1:6379> lrange list 0 -1
1) "1"
2) "2"
3) "3"
```

如果需要删除整个列表，可以使用 `del` 命令；如果需要获取列表长度，可以使用 `llen` 命令；判断列表是否存在，可以使用 `exists` 命令

```
127.0.0.1:6379> exists list
(integer) 1
127.0.0.1:6379> llen list
(integer) 3
127.0.0.1:6379> del list
(integer) 1
127.0.0.1:6379> llen list
(integer) 0
127.0.0.1:6379> exists list
(integer) 0
```

不可对其他类型的key进行操作。可以用 `type` 命令查看数据类型

```
> set foo bar
OK
> lpush foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```

## Unreadable Chinese

如果使用中文作为value，会出现不可读的问题（应该说是redis帮我们转码保存后，获取出来不能直观的显示出来）

```
127.0.0.1:6379> set test "我们"
OK
127.0.0.1:6379> get test
"\xe6\x88\x91\xe4\xbb\xac"
```

需要在启动时 redis-cli 后面加上 --raw

```
[root@VM_221_211_centos lib]# redis-cli --raw
127.0.0.1:6379> get test
我们
```

## Sets

Redis集合是一个无序的字符串合集。你可以以O(1) 的时间复杂度完成 添加，删除以及测试元素是否存在的操作。

和Java中Set一样，Redis集合不允许相同成员存在。向集合中多次添加同一元素，在集合中最终只会存在一个此元素。

一个Redis集合十分有趣的事是，它们支持从现有的多个集合中集合运算。 所以你可以在很短的时间内完成合并（union）,求交(intersection), 找出不同元素的操作。

使用 `SADD` 命令添加元素到集合中，可以发现是无序的(not sorted)；
使用 `SISMEMBER` 检查集合中某个元素是否存在

```
127.0.0.1:6379> sadd set 1 a 2 b 3 5
(integer) 6
127.0.0.1:6379> smembers set
1) "b"
2) "2"
3) "1"
4) "a"
5) "5"
6) "3"
127.0.0.1:6379> sismember set a
(integer) 1
```

Set对表达对象间的关系非常好，例如我们可以使用集合实现标签(tags)
一个方法就是对我们想要标记的对象建立Set。Set中包含与对象关联的标签的ID。

如下所示，ID或者名称为blog1的其中一篇blog被标记为Java、Linux和NetWork相关的标签

```
127.0.0.1:6379> sadd blogs:blog1:tags Java Linux NetWork
(integer) 3
```

这样我们就能够通过这篇blog的名称获得其标记的所有标签。

```
127.0.0.1:6379> smembers blogs:blog1:tags
1) "Linux"
2) "Java"
3) "NetWork"
```

我们有可能需要相反的关系所有的blog被同一个标签标记
如将blog1,blog2和blog3都标记为Java标记

```
127.0.0.1:6379> sadd tag:blog1:blogs Java
(integer) 1
127.0.0.1:6379> sadd tag:blog2:blogs Java
(integer) 1
127.0.0.1:6379> sadd tag:blog3:blogs Java
(integer) 1
```

使用 SPOP 命令获取第一个元素；使用 `SCARD` 查看元素个数(cardinality)

```
127.0.0.1:6379> spop blogs:blog1:tags
"NetWork"
```

## Hashes

这个我们就喜闻乐见了--我们期待已久的 field-value 键值对形式。
直接陈上用法
`HMSET` `HMGET` 分别用于设置和获取多个键值对，还有对应的`HSET`, `HGET`

下面的例子中是将一个ID为id1和id2的用户的用户名，生日和认证结果保存

```
127.0.0.1:6379> hmset user:id1 username u1 birth 1977 verified 1
OK
127.0.0.1:6379> hmset user:id2 username u1 birth 1989 verified 0
OK
127.0.0.1:6379> hget user:id1 username
"u1"
127.0.0.1:6379> hmget user:id1 username birth no-such-field
1) "u1"
2) "1977"
3) (nil)
```

可以通过 `HINCRBY` 等命令实现数值型字符串的计算

```
127.0.0.1:6379> hincrby user:id1 birth 5
(integer) 1982
127.0.0.1:6379> hget user:id1 birth
"1982"
```


## Sorted sets

有序集合允许我们根据自己的需要进行排序。像 Set 一样唯一不能重复，但是有序集合通过一个叫 `score`的值对本来无序的集合实现排序。
(score就不翻译了，中文官网翻译成'评分',我认为可以将'成绩'大小看成排序优先级高低)。

有序集合遵循以下原则：
1. 如果 A 和 B 具有不同的 score，那么如果 A.score > B.score，则 A > B
2. 如果 A 和 B 具有相同的 score，那么如果 A 字符串的字典排序大于(lexicographically greater) B 字符串的字典位置，则 A > B
3. A 和 B不可能相等因为 Set 中元素是唯一的

使用 `ZADD` 命令添加元素
下面的例子中，将 1940 等出生日期作为 hackers 这个key值的 score。用 `ZRANGE` 列出元素后发现元素已经排序，而排序规则正式根据 score 的大小升序排列(即生日大小)

可以使用 ZREVRANGE 对其进行降序

```
127.0.0.1:6379> zadd hackers 1940 "Alan"
(integer) 1
127.0.0.1:6379> zadd hackers 1957 "Sophie"
(integer) 1
127.0.0.1:6379> zadd hackers 1953 "Rechard"
(integer) 1
127.0.0.1:6379> zadd hackers 1949 "Anita"
(integer) 1
127.0.0.1:6379> zadd hackers 1965 "Yukihro"
(integer) 1
127.0.0.1:6379> zadd hackers 1914 "Hedy"
(integer) 1
127.0.0.1:6379> zadd hackers 1916 "Claude"
(integer) 1
127.0.0.1:6379> zadd hackers 1969 "Linus"
(integer) 1
127.0.0.1:6379> zadd hackers 1912 "Turing"
(integer) 1
127.0.0.1:6379> zrange hackers 0 -1
1) "Turing"
2) "Hedy"
3) "Claude"
4) "Alan"
5) "Anita"
6) "Rechard"
7) "Sophie"
8) "Yukihro"
9) "Linus"
```

使用 `withscores` 列出元素对应的 `score`；使用 `ZREMRANGEBYSCORE` 列出某 `score` 范围内的个数；使用 `ZRANK` 获取排序的位置(从0开始)。`ZRANGEBYSCORE` 列出某 `score` 范围内的元素。

对 `ZRANGEBYSCORE`命令，如果需要排除边界，可以加上 `(`
下面表示 1940< score <= 1960 的元素



```
127.0.0.1:6379> zrange hackers 0 -1 withscores
 1) "Turing"
 2) "1912"
 3) "Hedy"
 4) "1914"
 5) "Claude"
 6) "1916"
 7) "Alan"
127.0.0.1:6379> zrangebyscore hackers 1940 1960
1) "Alan"
2) "Anita"
3) "Rechard"
4) "Sophie"
127.0.0.1:6379> zrank hackers Turing
(integer) 0
127.0.0.1:6379> zrank hackers Claude
(integer) 2
127.0.0.1:6379> zrangebyscore hackers (1940 1960
1) "Anita"
2) "Rechard"
3) "Sophie"
...
```




### Lexicographical scores


测试数据：使 score 完全相同，可以观察到还是按照某种关系排序了

```
127.0.0.1:6379> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0 "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon" 0 "Linus Torvalds" 0 "Alan Turing"
(integer) 9
127.0.0.1:6379> zrange hackers 0 -1
 1) "Alan Kay"
 2) "Alan Turing"
 3) "Anita Borg"
 4) "Claude Shannon"
 5) "Hedy Lamarr"
 6) "Linus Torvalds"
 7) "Richard Stallman"
 8) "Sophie Wilson"
 9) "Yukihiro Matsumoto"
```



从[这里](http://redis.io/commands/zrangebylex)可以看到 score 是如何使用的。下面对其使用进行讲解，特别是**对中文的排序**。

我们需要知道的是对字符的比较是根据其二进制数组进行比较，对ASCII码来说，就是按字母顺序。但是如果存在非ASCII字符串，比如UTF-8的中文，会不准确。

对于其它字符集的比较可以使用如下方式
下面的例子中可以发现用户名的score都是0，但是姓氏有"林","廖","何"。此时我们不知道排序后的结果会这样(因为不可能是按照默认的二进制数组排序)。我们可以在value前加入真正需要排序的ASCII字符。
排列出来的结果就是按照he liao lin的结果排序。

注：需要在启动参数中添加--raw，具体查看[中文显示问题](redis-data-types#unreadable-chinese)

```
127.0.0.1:6379> zadd username 0 lin:"林"
(integer) 1
127.0.0.1:6379> zadd username 0 liao:"廖"
(integer) 1
127.0.0.1:6379> zadd username 0 he:"何"
(integer) 1
127.0.0.1:6379> zrange username 0 -1
he:何
liao:廖
lin:林
```


## Summary

Redis实际上是复杂的 KEY-VALUE形式存储的。怎么理解呢？ KEY 是字符串，而 Redis 的 data types 指的是后面的 VALUE，而这里的VALUE 可以只是简单的 value(String类型)，也可能有多个value(List类型, Set类型)，也可能是 key-value形式的(Hash类型)。

学习Redis，首先要明白 Redis 的数据类型，同时掌握基本的 Command，然后再掌握各种数据类型的使用场景。
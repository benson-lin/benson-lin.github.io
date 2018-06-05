---
title: "加锁同步"
layout: post
date: 2018-06-05
tags:
- Mysql
- PHP
categories: PHP
blog: true
excerpt: 在写代码过程中，经常会遇到需要加锁的情况，比如说请求的接口做了调用频率限制...
---

# 加锁

在写代码过程中，经常会遇到需要加锁的情况，比如说请求的接口做了调用频率限制，需要控制每个请求直接的频率，确保请求正常不会返回错误的响应。



单纯从前端看控制频率本身则是添加disabled不允许点击(如发送验证码一分钟只能发送一次)，但是后台要做到限制则需要加锁机制



我们可以利用redis的递增原子性来实现锁，确保每个请求直接是同步的



下面说下在开发中遇到的实际问题

1. 在一次操作中会涉及四个请求，四个请求用ABCD表示，调用顺序为A->B->C->D，其中接口方都是同一个服务提供者，它对总请求频率做了限制，也就是说不管ABCD哪个请求太快都会出问题
2. 可能有人会说要保证每个请求不要太快，在每个请求之前都sleep个几秒就好了。实际情况不止如此，会面临其他挑战，也就是第3点和第4点说的sleep和并发问题。
3. 在一次操作中，需要先进行AB过六分钟后才能进行CD操作(举个例子需要删除成功后才能做新增操作，但是删除操作本身是异步的，需要会几分钟才会有结果)，所以就会出现一次操作会出现停留很久的情况，所以可以认为会有这个过程A->B->sleep(5*60)->C->D。
4. 要进行A->B-->sleep(5*60)->C->D操作的频率比较高，也就是for循环中会出现多次调用的情况，但是for循环中每个卡5分钟以上是完全不可接受的，所以需要将每一次操作都加入队列(作为生产者)，同时去创建多个消费者去消费这些操作，从而避免了每次操作太久影响其他操作的问题，做到并行执行
5. 在加入并发后，会出现多个 A->B-->sleep(5*60)->C->D 同时进行的情况，接口方又有频率控制要怎么办呢，这时就可以加入全局锁的。在每次调用请求之前都去尝试获得锁，只有获得锁后才能发送请求
6. 代码类似如下，最后达到的效果是：
   - 每个操作之间不会互相干扰
   - 每个操作在sleep过程中能够给其他操作执行机会
   - 每个请求必须在另一个请求执行完成后才能执行另一个请求。每个请求之间会间隔10秒时间(也就是锁过期时间)



```php
getLock();
A
getLock();
B
sleep(5*60)
getLock();
C
getLock();
D
    
    
private function getLock()
{

    while(1){

        if (Redis::incr('my-lock') == 1) {
            Redis::expire('my-lock', 10);
            echo("* 获得锁");
            break;
        }
        sleep(2);
    }

}
```





如果会出现请求超时的问题，建议不要直接设置过期时间，这样可能还是会导致接口方接收到的请求过于频繁报错，我们可以在请求完成之后在释放锁。

其中有几个细节：

1. 获得锁之后还是需要设置过期时间，避免锁一直不释放，这个过期时间不是用来控制请求频率高低的。
2. 如果接口限制了每秒请求次数，可以在释放锁之前加入接口限制时间,sleep限制时间的描述



```php

try (
	getLock();
	A
) catch(\Exception $e) {
    
} finally {
    unsetLock();
}
try (
	getLock();
	B
) catch(\Exception $e) {
    
} finally {
    unsetLock();
}
try (
	getLock();
	C
) catch(\Exception $e) {
    
} finally {
    unsetLock();
}
try (
	getLock();
	D
) catch(\Exception $e) {
    
} finally {
    unsetLock();
}

    
private function getLock()
{

    while(1){

        if (Redis::incr('my-lock') == 1) {
            Redis::expire('my-lock', 60);
            echo("* 获得锁");
            break;
        }
        sleep(2);
    }

}

private function unsetLock()
{
	if (Redis::exists('my-lock')) {
	    sleep(3);
        Redis::del('my-lock');
	}
}
```


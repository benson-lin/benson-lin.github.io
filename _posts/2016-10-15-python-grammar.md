---
title: "Python Basis 01"
layout: post
date: 2016-10-15
tags:
- Python
categories: PYTHON
excerpt: Python基础语法介绍
---

- Python是**解释性语言**，在运行python程序时解析程序行；相比C或C++，它们先进行编译，然后再运行
- Python是**动态类型语言**，会基于变量值，自动的判断变量的数据类型；
- Python是**强类型语言**, 变量在类型决定下来以后(即使是动态判断数据类型)，以后只能是那几种相关类型

本博客将在以后的学习中不断补充下面的知识点有但是未在文章中展示的内容

## Text input and output

注意是没有分号的

```python
#输出
print("Hello World")
print("Hello World\nThis is a message")

x = 3
print(x)

x = 2
y = 3
print(x, ' ', y)
```

```python
#输入
i = input()
print(x)
```

## Command line arguments

控制台输入参数

```python
import sys

print('Arguments:', len(sys.argv))
print('List:', str(sys.argv))


$ python3 tmp.py image.bmp color
Arguments: 3
List: [‘tmp.py’, ‘image.bmp’, ‘color’]
```

```python

from sys import argv

script, first, second, third = argv
print "The script is called:", script
print "Your first variable is:", first
print "Your second variable is:", second
print "Your third variable is:", third

#python ex13.py first 2nd 3rd
```

## If statements

- if 后面的冒号可以认为是C语言中的大括号
- 需要将代码放到py中,直接在控制台会将空行后的回车作为程序结束标志，所以控制台的代码不能有空行

```python
x = int(input("Tell X"))

if x == 4:
    print('You guessed correctly!')
elif x == 5 or x==6:
	print('Close to the right answer!')
else:
    print('Wrong guess')

print('End of program.')
```

使用 not 和 and(or), 下面专门不用!=，而是使用not

```python
x = int(input("Tell X: "))
y = int(input("Tell Y: "))

if not (y == 8 and x == 8):
    print('Wrong guess')
else:
    print('You guessed correctly!')

print('End of program.')
```

## For loops/While loop

- range 函数为 `a<=x<b`，不包含右边界
- python 中没有`do..while`结构

下面打印1-10

```python
for i in range(1,11):
    print(i)
```

```python
x = 0
while x != 5:
   x = int(input("Guess a number:"))
   
   if x != 5:
      print("Incorrect choice")

print("Correct")
```

循环是让计算机做重复任务的有效的方法，有些时候，如果代码写得有问题，会让程序陷入“死循环”，也就是永远循环下去。这时可以用Ctrl+C退出程序，或者强制结束Python进程。


## List

能够包含文本，整数，浮点数等。

```python
L = [] #允许定义空的list，长度为0
c = [5,2,10,48,32,16,49]
print(c[0])
print(c[-1]) # list最后一个的值,同理，c[-2]是倒数第二个元素
print(len(c))

# append 追加到list末尾
c.append(100)
print(c) #输出[5, 2, 10, 48, 32, 16, 49, 100]

# insert 插入到指定位置
c.insert(1, 0); # 第一个参数是位置，第二个参数是值
print(c) #输出[5, 0, 2, 10, 48, 32, 16, 49, 100]

# 使用pop()删除最后一个元素，
c.pop()
print(c) #输出[5, 0, 2, 10, 48, 32, 16, 49]

# 使用pop(i)删除特定位置元素
c.pop(1)
print(c) #输出[5, 2, 10, 48, 32, 16, 49]

# 使用sort排序
c.sort()
print(c) #输出[2, 5, 10, 16, 32, 48, 49]

```

如果下标超过 数组长度-1，解释器报错

```
print(c[len(c)])
IndexError: list index out of range
```

list里面的元素的数据类型也可以不同

```python
L = ['Apple', 123, True]
```

ist元素也可以是另一个list

```python
s = ['python', 'java', ['asp', 'php'], 'scheme']
print(len(s)) #输出4
```

PYTHON 还支持切片功能，不用循环，直接获取 list 或 string 或 tuple 中某个范围内的元素.

```python
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
print(L) 
 
#before
r = []
n = 3
for i in range(n):
    r.append(L[i])
print(r)

print("-"*10)

#now
print(L[0:3])  #从索引0开始取，直到索引3为止，但不包括索引3
print(L[:3])   #如果第一个索引是0，还可以省略
print(L[1:3])  #从索引1开始，取出2个元素
print(L[-2:])  #支持倒数切片
print(L[-2:-1]) #同样不包括右边界
print(L[:])         #只写[:]就可以原样复制一个list
print(L[::2])#每2个取一个

print("-"*10)

print((0, 1, 2, 3, 4, 5)[:3]) #tuple也支持
print('ABCDEFG'[:3]) #string也支持
```



## Tuple

tuple和list非常类似，但是tuple一旦初始化就不能修改，因此没有list的insert和append方法，也不能对某个下标的值进行修改，只能进行读取

使用()括起来。

```python
names = ('Michael', 'Bob', 'Tracy')
```

因为tuple不可变，所以代码更安全。如果可能，能用tuple代替list就尽量用tuple。

当你定义一个tuple时，在定义的时候，tuple的元素就必须被确定下来

```python
t1 = () # 创建一个空tuple
t2 = (11,22,33)
t3 = tuple([1,2,3,4,4]) # 从list中构造tuple
t4 = tuple("abc") # 从string构造tuple
t6 = (34,)#注意定义一个元素时，末尾有个逗号

#如果用print输出，结果如下
()
(11, 22, 33)
(1, 2, 3, 4, 4)
('a', 'b', 'c')
(34,)
```

为甚么不能直接 `t6 = (34)`，因为()既可以表示tuple，又可以表示数学公式中的小括号，这就产生了歧义，因此，Python规定，这种情况下，按小括号进行计算，计算结果自然是1。

## Dictionary

- key-value形式；dict的key必须是不可变对象；在Python中，字符串、整数等都是不可变的，因此，可以放心地作为key。而list是可变的，就不能作为key
- 其实就是Java中的Map；
- 使用del删除元素
- 如果key不存在，dict就会报错。要避免key不存在的错误，有两种办法，一是通过in判断key是否存在；二是通过dict提供的get方法，如果key不存在，可以返回None，或者自己指定的value
- 也可以使用dict.pop(key)方法删除一个key，对应的value也会从dict中删除

```python
k = { 'EN':'English', 'FR':'French' }
print(k['EN'])

k['DE'] = 'German'

k = { 'EN':'English', 'FR':'French' }
del k['FR']
print(k)

print("EN" in k)

print(k.get("CHN"))
print(k.get("CHN", "CHINA"))#这个KEY不会添加到dict中

k.pop("EN")
print(k)
```

输出

```python
English
{'EN': 'English'}
True
None
CHINA
{}
```

## Set

- 传入一个list, 重复元素在set中自动被过滤：
- 通过add(key)方法可以添加元素到set中，可以重复添加，但不会有效果
- 通过remove(key)方法可以删除元素
- set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作

```python
s = set([1, 2, 3, 1, 2, 4])
print(s)

s.add(0)
s.add(4)
print(s)
```

输出

```python
{1, 2, 3, 4}
{0, 1, 2, 3, 4}
```


set的原理和dict一样，所以，同样不可以放入可变对象，因为无法判断两个可变对象是否相等，也就无法保证set内部“不会有重复元素”

## Functions

对重复的代码，可以使用函数, 函数可以添加返回值

```python
import math

def pythagoras(a,b):
    value = math.sqrt(a *a + b*b)
    print(value)

pythagoras(3,3)
```
使用return

```python
import math

def pythagoras(a,b):
    value = math.sqrt(a*a + b*b)
    return value
    
result = pythagoras(3,3)
print(result)
```

**可以返回多个值**，挺不错的；但其实这只是一种假象，Python函数返回的仍然是单一值

```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny

# x, y = move(100, 100, 60, math.pi / 6) 外部这样赋值
# r = move(100, 100, 60, math.pi / 6) # 输出(151.96152422706632, 70.0)
```

原来**返回值是一个tuple**！但是，在语法上，返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值，所以，Python的函数返回多值其实就是返回一个tuple

可以为函数**添加默认参数值**，像c++中的构造函数一样

```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
# power(5) 和 power(5, 2)就不会报错了
```

设置默认参数时，**必选参数在前，默认参数在后**，否则Python的解释器会报错（思考一下为什么默认参数不能放在必选参数前面）；

对不符合默认参数顺序的调用，需要显示设置参数值，例子如下

```python
def enroll(name, gender, age=6, city='Beijing'):
    print 'name:', name
    print 'gender:', gender
    print 'age:', age
    print 'city:', city
# enroll('Bob', 'M', 7)
# enroll('Adam', 'M', city='Tianjin') 显式设置city
```

为什么要设计str、None这样的不变对象呢？因为不变对象一旦创建，对象内部的数据就不能修改，这样就减少了由于修改数据导致的错误。此外，由于对象不变，多任务环境下同时读取对象不需要加锁，同时读一点问题都没有。我们在编写程序时，如果可以设计一个不变对象，那就尽量设计成不变对象。

还可以设置**可变参数**

```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
定义可变参数和定义list或tuple参数相比，仅仅在参数前面加了一个*号。在函数内部，参数numbers接收到的是一个tuple，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括0个参数：

```python
>>> calc(1, 2)
5
>>> calc()
0
```
如果已经有一个list或者tuple，要调用一个可变参数怎么办？可以这样做

```python
>>> nums = [1, 2, 3]
>>> calc(*nums)
14
```

还可以定义**关键字参数**。可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict

```python
def person(name, age, **kw):
    print 'name:', name, 'age:', age, 'other:', kw
```
```python
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```



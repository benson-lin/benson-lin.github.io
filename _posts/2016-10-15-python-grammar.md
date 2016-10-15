---
title: "Python Grammar"
layout: post
date: 2016-09-24
permalink : /post/python=grammer
tag:
- Python
blog: true
---


## Text input and output

注意是没有分号的；
```python
print("Hello World")

print("Hello World\nThis is a message")

x = 3
print(x)

x = 2
y = 3
print(x, ' ', y)
```

## If statements

- if 后面的冒号可以认为是C语言中的大括号
- 需要将代码放到py中,直接在控制台会将空行后的回车作为程序结束标志，所以控制台的代码不能有空行

```python
x = int(input("Tell X"))

if x == 4:
    print 'You guessed correctly!'
else:
    print 'Wrong guess'

print 'End of program.'
```


## For loops

- range 函数为 a<=x<b，不包含右边界
下面打印1-10

```python
for i in range(1,11):
    print i
```

## While loop

```python
x = 0
while x != 5:
   x = int(input("Guess a number:"))
   
   if x != 5:
      print("Incorrect choice")

print("Correct")
```


## Functions

对重复的代码，可以使用函数
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

## Lists

能够包含文本，整数，浮点数等。

```python
c = [5,2,10,48,32,16,49,10,11,32,64,55,34,45,41,23,26,27,72,18]
print(c[0])
print(c[1])
print(c[-1])
print(len(c))
fears = ["Spiders","Ghosts","Dracula"]
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

## Dictionary

key-value形式；
其实就是Java中的Map；
del删除元素

```python
k = { 'EN':'English', 'FR':'French' }
print(k['EN'])

k['DE'] = 'German'

k = { 'EN':'English', 'FR':'French' }
del k['FR']
print(k)
```

## Read and Write File

读写文件
```python
filename = "readme.txt"
 
with open(filename) as fn:
    content = fn.readlines()
 
print(content)
```

```python
f = open("output.txt","w")
f.write("Pythonprogramminglanugage.com, \n")
f.write("Example program.")
f.close()
```

## Statistics

统计；模块需要自己下载
```python
import math
import numpy

x = [1,2,15,3,6,17,8,16,8,3,10,12,16,12,9]

print(numpy.min(x))
print(numpy.max(x))
print(numpy.std(x))
print(numpy.mean(x))
print(numpy.median(x))
```

```python
import matplotlib.pyplot as plt
import numpy as np

x = [1,2,15,3,6,17,8,16,8,3,10,12,16,12,9]

plt.boxplot(x)
plt.show()  
```

## Line charts

折线图

默认是蓝色，最后一行修改为红色加点

```python
import numpy as np
import matplotlib.pyplot as plt

x = [2,3,4,5,7,9,13,15,17]
plt.plot(x)
plt.ylabel('Sunlight')
plt.xlabel('Time')
plt.show()

plt.plot(x, 'ro-') 
```

## Modules

用于组织代码
模块是python中的一或多个 函数或者变量，能够被调用通过import

```python
# hello.py
def hello():
    print("Hello World")
```
```python
# Import your module
import hello

# Call of function defined in module
hello.hello()
```

查看所有变量

```python
import math

content = dir(math)
print(content)
```

## Date and Time

- 计算机使用ticks处理时间，所以计算机从12:00am, January 1, 1970开始计时，作为大纪元时间
- 使用 time 模块
- 大纪元时间time和当前机器时间localtime
- 当前时间默认为未格式化，需要用数组下标取出
- 使用asctime输出格式化的时间。

```python 
import time 

# Epoch time
ticks = time.time()
print "Ticks since epoch:", ticks
# Ticks since epoch: 1461118370.5304039

# Local time
timenow = time.localtime(time.time())
print "Current time :", timenow
# Current time : time.struct_time(tm_year=2016, tm_mon=4, tm_mday=20, tm_hour=10, tm_min=13, tm_sec=51, tm_wday=2, tm_yday=111, tm_isdst=0)

print "Year:", timenow[0]
print "Month:", timenow[1]
print "Day:", timenow[2]
# Year: 2016
# Month: 4
# Day: 20

timenow = time.asctime(time.localtime(time.time()))
print(timenow)
# Wed Apr 20 10:13:51 2016
```

## Comments

- 单行注释：#comment
- 多行注释：'''comment'''

```python
# This is a comment
# second line
print('Hello')

''' This is a multiline 
Python comment example.'''
x = 5
```

## Strings and Sub-strings

```python
s = "Hello World"
print(s)
print(s[0])

# String Slicing 字符串片，一样是a<=x<b
print(s[:3]) ##不包含下标3
print(s[3:]) 
print(s[3:6])

# Hel
# lo World
# lo

slice = s[0:5]
```

## Regular expressions

- in | no in
- import re
- \d 	Matches a decimal digit; equivalent to the set [0-9].
- \D 	The complement of \d. It matches any non-digit character; equivalent to the set [^0-9].
- \s 	Matches any whitespace character; equivalent to [ \t\n\r\f\v].
- \S 	The complement of \s. It matches any non-whitespace character; equiv. to [^ \t\n\r\f\v].
- \w 	Matches any alphanumeric character; equivalent to [a-zA-Z0-9_]. 
- \W 	Matches the complement of \w.
- \b 	Matches the empty string, but only at the start or end of a word.
- \B 	Matches the empty string, but not at the start or end of a word.
- \\ 	Matches a literal backslash.
```python
s = "Are you afraid of ghosts?"

print("ghosts" in s)
# True

print("coffee" not in s)
# True
```

```python
import re

s = "123"
matcher = re.match('\d{3}\Z',s)

if matcher:
    print("True")
else:
    print("False")
```


## Python Socket Client (Network)

流程和平时的socket连接流程是一样的

- create socket
- get server ip address from domain name
- connect to server using ip address
- send request to server
- receive data (webpage)



```python
import socket
import sys  

host = 'https://www.baidu.com'
port = 80 # web
 
# create socket
print('# Creating socket')
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error:
    print('Failed to create socket')
    sys.exit()
     
 
print('# Getting remote IP address') 
try:
    remote_ip = socket.gethostbyname( host )
except socket.gaierror:
    print('Hostname could not be resolved. Exiting')
    sys.exit()
 
# Connect to remote server
print('# Connecting to server, ' + host + ' (' + remote_ip + ')')
s.connect((remote_ip , port))
 
# Send data to remote server
print('# Sending data to server')
request = "GET / HTTP/1.1\r\n\r\n"
 
try:
    s.sendall(request.encode())
except socket.error:
    print('Send failed')
    sys.exit()
 
# Receive data
print('# Receive data from server')
reply = s.recv(4096)
 
print(reply) 


# Creating socket
# Getting remote IP address
# Connecting to server, www.baidu.com (14.215.177.37)
# Sending data to server
# Receive data from server
# b'HTTP/1.1 302 Moved Temporarily\r\nDate: Wed, 20 Apr 2016 02:49:31 GMT\r\nContent-Type: text/html\r\nContent-Length: 215\r\nConnection: Keep-Alive\r\nLocation: http://www.baidu.com/search/error.html\r\nServer: BWS/1.1\r\nX-UA-Compatible: IE=Edge,chrome=1\r\nBDPAGETYPE: 3\r\nSet-Cookie: BDSVRTM=0; path=/\r\n\r\n'
```

## Socket Server (Network)

- bind socket to port
- start listening
- wait for client
- receive data

要测试，只需要修改上面的host和port

```python
import socket
import sys
 
HOST = ''  
PORT = 7000
 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
print('# Socket created')
 
# Create socket on port
try:
    s.bind((HOST, PORT))
except socket.error as msg:
    print('# Bind failed. ')
    sys.exit()
     
print('# Socket bind complete')
 
# Start listening on socket
s.listen(10)
print('# Socket now listening')
 
# Wait for client
conn, addr = s.accept()
print('# Connected to ' + addr[0] + ':' + str(addr[1]))

# Receive data from client
while True:     
    data = conn.recv(1024)
    line = data.decode('UTF-8')    # convert to string (Python 3 only)
    line = line.replace("\n","")   # remove newline character
    print( line )     
     
s.close()
```
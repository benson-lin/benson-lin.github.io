---
title: "Alibaba Interview (Java): Part Three & Part Four"
layout: post
date: 2016-04-16
permalink : /post/alibaba-interview-java-part-three-and-four
tag:
- INTERVIEW
blog: true
---

## 应聘岗位

研发工程师Java（2016/03/26 & 2016/03/29）

## 面试题

第三和第四面都是线下做题，要求不允许查阅资料，在规定时间自行实现

### 第三面

时间：2 hour

26号中午刚刚吃完饭，差不多到12点，面试官打过来，要求在下午2点前将下面这道题做出来。

```java
//IP地址库查找，自定义内存中的数据结构，需要考虑性能最优，内存最少。
//精确编写该类的实现并且包括单元测试程序。
//编译环境为标准Java。
public class IpLib{

	//第一个接口
	 /* file 为IP地址库文件，格式为每行一个点分十进制的IP，上亿条。 需要判断IP格式是否正确，且IP需要去重 */
	bool LoadIpLibFile( String file);

	//第二个接口	
	/* 输入一个点分十进制的IP，若该IP在内存数据结构中，返回true,否则返回 false */
	bool Find( String ip); 
｝
```

我当时想到的方法 创建1亿个元素的long数组，然后将每一个读入的IP用正则判断无误后转为 long 放入数组中，然后用快速排序排序，对输入的IP转long进行二分查找。当然去重操作是做不到的。后来时间不够了，就发给面试官了。


### 第四面

时间：4 hour

29号晚上8点打过来将我做下面这道题，专门提出考察学习能力，晚上11点将程序发给他。
后来做不完，他打电话过来给多我一次机会，时间为1个小时，11:15开始做，12:15分还是做不完，就把修改了的程序发给面试官了。
当时感觉挂了，第二天居然HR打过来了，我程序没跑成功的啊！



考察学习能力：

> 阅读 RFC2616 文档，即 HTTP/1.1 规范，输入某个网址，利用 Java 的 Socket 发送 HTTP请求，特别要求能够解码 chunked 编码，观察文档中的伪代码实现，自己用Java代码实现，将解析后的整个html文档输出到控制台上，不要求关注太多细节。(就是不允许用httpclient的jar包，自行实现这个jar包类似的功能)
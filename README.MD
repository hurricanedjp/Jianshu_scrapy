# Jianshu 简书爬虫与用户关系网络分析
1. 爬取简书(http://www.jianshu.com/)  的用户数据

   （包括 ：uid，昵称，头像，性别，关注数，粉丝数，文章数，字数，获得的赞的数目）
2. 爬取用户之间的following关系
3. 进行用户的数据分析，关系网络的分析


## 分析
1. 爬虫使用的是scrapy爬虫框架

    1.1 jianshu 的很多数据使用的是js在动态加载，使用scrapy爬去动态的数据

    1.2 特意为爬虫写了一个ip的代理池，这样可以绕过ip的限制（https://github.com/zhaozhengcoder/Ip_pool)
2. 数据库用的是关系型数据库mysql （目前爬去了8w条用户信息，和25w的用户关系）
3. 下一步讲关系型数据库的数据导入图数据库NEO4J，这样进行关系网络的分析的时候，会方便一点


## Todo：
1. 爬虫五个小时爬去了8w条用户信息和25w条用户关系，下一步打算进一步学习一个scrapy框架，再进一步提高一下爬取的效率
2. 改变一下现有的爬取策略，现有的用户的信息有较强的局部性，争取能够爬取到更加完整，全面的用户信息
3. 使用图数据库进行用户的关系网络分析


## Other 
```
Scrapy  : 1.0.3
lxml    : 3.5.0.0
libxml2 : 2.9.3
Twisted : 16.0.0
Python  : 2.7.12 (default, Nov 19 2016, 06:48:10) - [GCC 5.4.0 20160609]
Platform: Linux-4.4.0-97-generic-x86_64-with-Ubuntu-16.04-xenial

爬虫是一个强依赖于网站的html结构的，如果网站出现前端版本更新，那么会导致爬虫解析html的policy出问题。
截止到2017年10月31日，程序是没有问题的，之后我们维护一段时间。再之后，就不一定了。
毕竟，人生苦短，我还要和女朋友一起玩耍～ 
逃～
```


## 后续的思考
爬虫爬下来的数据存在mysql里面，主要的数据是两大块，一是：作者的信息，二是：作者之间的following关系。这两块信息存储的时候，设计了两张表：

第一张表存储作者的信息
```
（uid ，姓名，性别，文章数，被喜欢的次数，关注的用户数，粉丝数，字数，头像的url）
```
第二张表存储作者的follow关系
```
（uid，following_uid） #这个表示uid对应的用户 的一个粉丝是following——uid对应的用户
```

这样看着还行，但是有一个问题是，割裂了用户信息和用户粉丝信息，把他们分成了两张表。如何涉及到这样的查询的时候，就必然需要大量的join操作。

在github上面看见这位大哥的分析，很好 ：https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/MongoDB%20%E6%9E%81%E7%AE%80%E5%AE%9E%E8%B7%B5%E5%85%A5%E9%97%A8.md

简单的说，使用monogdb 可以解决这种文件，它可以把所有的信息堆在一张长得很像json的表上。刚才说的mysql上面存在的问题，就轻松解决！！！！　

so,　最近估计会重构一个mongodb的版本!!!

### update :
找了一个不忙的半天，学习了一些mongodb的语法，更新了一个使用mongodb的版本

### 2017.11.02  update :
1. 发现了一个问题，就是自己没有写去重的policy，下一步可能会增加去重的策略，提高爬虫的效率

2. 想学习一个分布式的爬虫ｏｒ多线程爬虫　，提高一下效率


### update :
关于实现重复爬取页面去重的策略的想法，

１．使用set, 将爬过的url存放在set里面，对于一个newurl，进行一个判断　
```
    if newurl in urlset:
        不爬取这个url
    else 
        爬取这个url
```
２．使用bloomfilter
```
#自己实现了一个布隆过滤器，在可以在目录里面看见这个文件
myBloomFilter.py
```
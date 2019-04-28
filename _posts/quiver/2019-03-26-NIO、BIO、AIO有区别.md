---
layout: post
title: NIO、BIO、AIO有区别
date: 2019-03-26
tags:
   - io
   - java 
---
#BIO
> Blocking IO 同步阻塞IO,在刚学习socket时用的io是bio

- 同步: 是否要亲自去监听操作，不断进行关注，是否已经完成对应操作。
- 阻塞：io操作如果没有完成，在单线程的环境下，其他客户端是不能够连接上。

解决阻塞问题：对于创建多线程去处理
存在问题：针对多个客户端的连接，服务端的纯种数据量肯定是会增加的。
   1. 创建线程只发生io的时候。
   2. 
多次开启与关闭
# NIO（JDK1.4），引用chanal 与selector\buffer
> New IO(No-Blocking IO), 同步非阻塞
- 同步
- 非阻塞

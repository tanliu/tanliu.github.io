---
layout: post
title: TCP、HTTP、HTTPS
date: 2019-04-08
tags:
   - 网络
   - http
   - tcp
---
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/539B465A7785885AEB82C085773EE711.jpg)

tcp头：目标端口号
ip头：目前ip地址、源ip地址


3.深入分析tcp/ip
- DNS解析
- 网关进行数据传输
- 路由协议
- 握手协议（三次握手、四次挥手）
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/63A69E326A441B60D40E7511DD9CA1A3.jpg)
  - 为什么要三次握手、两次握手可不可以？
     两次握手会有一下问题。当client 发起请求c1时，由于网络情况，c1还没有收到Server回应，以为请求丢失了（其实是网络延迟），然后再次发出了c2连接请求。server最终可能会收到两个连接请求。这样就会有两个连接。引用三次握手的解决无效请求创建连接，造成资源浪费的问题。
  - 为什么是四次挥手，三次可不可以？
     三个会有一个问题。最终server回关闭服务器后，发出的请求。client没有收到。造成client无法关闭。
- TCP流量控制（滑动窗口）
  - https://coolshell.cn/articles/11609.html



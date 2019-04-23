---
layout:     post
title: zookeeper的实现原理
date: '2018-08-22'
tags:
- zookeeper
---
##  数据流
1.读请求，可以到任何节点
2.写请求，最终会到leader中进行同步
![IMAGE](/images/q/IMAGE) 

- Follower节点收到客户端写请求
- Follower把写请求转发给Leader
- Leader向所有Follower广播proposal（Leader在发前proposal前会生成提一个全局唯一的ID,同时leader会为每个Follower生成一个FIFO的队列以保证发给每个follower的事务都有序的）
- Follower收到proposal后，回复ack
- Leader收到合法数量（半数）的ack后，自己commit事务后，向Follower发送commit请求。

## 消息广播


## ZAB协议
> zab（Zookeeper Atomic Broadcate）支持崩溃恢复的原子广播协议，协议主要是保证数据一致性、
**关于zab事务处理**:所有事务是由leader管理，其它节点是follower，leader向所有follower发起proposal，当超过半数follower回复，则leader向follower发起commit请求。

### 崩溃恢复
在启动、崩溃重启、网络中断等异常情况下。网络导致leader与超过半数提follower失去联系，zab会进入恢复模式并选举出新的leader,当过半的follower与新leader完成过半的数据同步后，zab会退出恢复模式。
1. 当leader失去过半Follower的联系
2. Leader结点挂了


### 消息广播
当过半的follower与新leader完成过半的数据同步后，系统框架就可以进入消息广播了。些时，如果就新follower加入，新follower自动找到leader，并进入数据恢复模式

### 原子广播提实现

### ZXID 的原理
> 是由64位数字组成，前32位是leader的epoch,后三十位是proposal的自增
> 当选举出新leader时，会获取最大的proposal的zxid，前32位增1


## CAP原理
一致性（Consistency）:
可用性（Availability）:
分区容错性（Partition tolerance）:

## BASE理论

## 2pc原理
>2pc（2 phace commint）二阶提交。集群必需要有leader与follower

### 2pc的流程
第一阶段，事务操作，具体流程如：

1. Leader向Follower发送事务请求，Follower决定undo或者redo记录到事务日志中
2. Follower给Leader回复（YES or NO）


第二阶段，事件提交
1.如果Follower回复Yes，那么Leader会给每个Follower发送commit,Follower回复ack
2.如果Follower回复NO,那么Leader会向Follower发送rollback请求

![IMAGE](/images/q/IMAGE)
### 优点&缺点
- 优点：原理比较简单，容易实现
- 缺点：同点阻塞、单点问题、脑裂、太过保守
  - 同步阻塞
  - 单点问题：leader死了，什么也干不了了
  - 脑裂：发commint发出一半后，leader挂了


## paxos算法
> 角色：  Proposer、Acceptor、 Learner

恶心情况：
  1.提案前机器重启或者失败，提案后重启或者失败
  2.传输过程中出现的不可以控的情况，如：网络延迟


推导过程：












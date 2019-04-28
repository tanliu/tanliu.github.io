---
layout: post
title: zookeeper部署与配置详解
date: 2018-07-31
tags:
   - zookeeper
---
### zookeeper环境配置
1.单机环境配置
   - 参考 [http://zookeeper.apache.org/doc/r3.4.11/zookeeperStarted.html#ch_GettingStarted]

2.集群环境配置
   ```
    tickTime=2000 //时间的最小单元
    dataDir=/var/lib/zookeeper
    clientPort=2181  //客户端连接的端口
    initLimit=5  //？
    syncLimit=2  //？
    server.1=zoo1:2888:3888  //ip一下集群端口，一下选举端口？
    server.2=zoo2:2888:3888
    server.3=zoo3:2888:3888
    
    配置 $dataDir/myid下的id
   ```

### zookeeper的结果
> zookeeper的数据结构与文件系统的大体相同，不同之处是znode可以存放一些数据，同时是有顺性的

1.znode数据结构
```
cZxid = 0x100000009  //创建时的版本号
ctime = Tue Jul 31 13:26:06 HKT 2018  //创建的时间
mZxid = 0x10000000a //最后修改版本（modified）
mtime = Tue Jul 31 13:26:28 HKT 2018
pZxid = 0x100000009  //孩子节点最后修改版本
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 11
numChildren = 0
```



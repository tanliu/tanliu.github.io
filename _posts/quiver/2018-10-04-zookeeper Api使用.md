---
layout: post
title: zookeeper Api使用
date: 2018-10-04
---
#### 数据的增删改查操作（test）
```
public static void main(String[] args) {
        final CountDownLatch countDownLatch = new CountDownLatch(1);

        try {
            ZooKeeper zooKeeper = new ZooKeeper("172.26.35.30:2181", 50000, new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    if (Event.KeeperState.SyncConnected == watchedEvent.getState()){
                        countDownLatch.countDown();
                    }else {
                        System.out.println(watchedEvent.getState());
                    }

                }
            });
            countDownLatch.await();
            System.out.println(zooKeeper.getState());
//            zooKeeper.delete("/tanliu/test1",2);
            zooKeeper.create("/tanliu/test","tanliu".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

            //节点基本信息
            Stat stat = new Stat();
            //获取当前的值
            byte[] data = zooKeeper.getData("/tanliu/test", null, stat);
            System.out.println(new String(data));

            //修改节点的值
            zooKeeper.setData("/tanliu/test","tanliu_test".getBytes(),stat.getVersion());


            //删除节点
            zooKeeper.getData("/tanliu/test", null, stat);
            zooKeeper.delete("/tanliu/test",stat.getVersion());
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }


    }
```
#### 事件机制（Watch）
1. 当数据发生变化后，zookeeper会创建一个事件发送到订阅客户端。但是订阅只会生产一次。
2. 事件订阅入口
   - getData
   - exists
   - getChildren
3. 事件的触发
   - delete
   - create
   - setData
4. 事件类型
    - None (-1), 客户端连接状态发生变化
    - NodeCreated (1), 创建节点的事件
    - NodeDeleted (2), 删除节点的事件
    - NodeDataChanged (3), 节点数据发生变更事件
    - NodeChildrenChanged (4); 子节点发生变更（增删改查）的事件



### 事件的原理
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/9C5FD7E42BB0FCBB770C2501E800285C.jpg)

### 查询节点








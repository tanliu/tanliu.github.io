---
layout: post
title: Zookeeper源码
date: 2018-08-31
tags:
   - zookeeper
---
部署的方法：
https://segmentfault.com/a/1190000016078831


###
### 关于源码阅读的一些心得
1. 所有的配置都是在开始已经初始化已经加载完的
   eg:不知道用哪个实现类的时候，可以回到 init 阶段找
2. 判断是流程的走向

### zookeeper的watch大致原理
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/16E96481C411AF85302700F547C2A856.jpg)
### 创建连接：



    <script src="http://code.jquery.com/jquery-1.4.2.min.js"></script>
    <script src="../.resources/raphael-min.js"></script>
    <script src="../.resources/underscore-min.js"></script>
    <script src="../.resources/sequence-diagram-min.js"></script>
    <script src="../.resources/flowchart.min.js"></script>
    <div class="sequence">Title: 创建连接的流程
Note over Zookeeper: defaultWatchManager() \n 创建默认watchManager
Note over Zookeeper: createConnection（） \n 创建连接
Zookeeper -> ClientCnxn : start() 启动两个线程: \n sendThread.start(); \neventThread.start();
</div>

```
源码如下：
```
    public ZooKeeper(String connectString, int sessionTimeout, Watcher watcher,
            boolean canBeReadOnly, HostProvider aHostProvider,
            ZKClientConfig clientConfig) throws IOException {
        LOG.info("Initiating client connection, connectString=" + connectString
                + " sessionTimeout=" + sessionTimeout + " watcher=" + watcher);

        if (clientConfig == null) {
            clientConfig = new ZKClientConfig();
        }
        //创建一个默认的clientConfig
        this.clientConfig = clientConfig;
        //创建一个默认的watchManager
        watchManager = defaultWatchManager();
        watchManager.defaultWatcher = watcher;
        ConnectStringParser connectStringParser = new ConnectStringParser(
                connectString);
        hostProvider = aHostProvider;

        //创建建连接（应该创建连接信息）
        cnxn = createConnection(connectStringParser.getChrootPath(),
                hostProvider, sessionTimeout, this, watchManager,
                getClientCnxnSocket(), canBeReadOnly);
        //开始连接，启动了两个线程
        //       sendThread.start();
        //        eventThread.start();
        cnxn.start();
    }
```


```
## Watcher原理
### 客户端

<div class="sequence">Note over Zookeeper: exists()
Zookeeper -> ClientCnxn:submitRequest（）
Note over ClientCnxn: queuePacket()\n 把请求放入到outgoingQueue中 

Note over SendThread: doTransport()处理queue里面\n的信息基中doTransport有两个实现类\n分别是\nClientCnxnSocketNIO 和ClientCnxnSocketNetty
</div>
1. 调用exists方法
2. 把数据信息打包放入到queue中
3. sendThread 和 eventThread 处理queue中的内容

### 服务端

<div class="sequence">Note over NettyServerCnxn: 1.receiveMessage（）\n 接收客户端的数据
NettyServerCnxn -> ZooKeeperServer: 2.processPacket()
ZooKeeperServer -> ZooKeeperServer: 3.submitRequest()
ZookeeperServer -> PrepRequestProcessor: 4.processRequest()
PrepRequestProcessor -> SyncRequestProcessor: 5.processRequest()
SyncRequestProcessor -> FinalRequestProcessor:  6.processRequest()
FinalRequestProcessor -> ZooKeeperServer: 7.sendResponse()
</div>
1.接收到客户端的信息
2.把头信息从byteBuffer中解出，封装起来
3.submitRequest把信息放到RequestProcessor链式中得处理
4.PrepRequestProcessor把信息放入到submittedRequests进行异步处理
5.SyncRequestProcessor把信息入入到queuedRequests 异步处理
6.FinalRequestProcessor最后消息的处理
7.FinalRequestProcessor调用ZooKeeperServer的sendResponse消息。
<div class="sequence">
</div>



                    
<script>
$(".sequence").sequenceDiagram({theme: 'simple'});
</script>                    

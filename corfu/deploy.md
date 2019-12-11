 # 服务器启动后进行配置时的行为

## 1. 收到layout

```txt
worker-0 |   o.c.i.ServerHandshakeHandler | channelActive: Incoming connection established from: /127.0.0.1:40448 Start Read Timeout.
worker-0 |   o.c.i.ServerHandshakeHandler | channelRead: Handshake Message received. Removing readTimeoutHandler from pipeline.
worker-0 |   o.c.i.ServerHandshakeHandler | channelRead: node id matching is not requested by client.
worker-0 |   o.c.i.ServerHandshakeHandler | channelRead: Handshake validated by Server.
worker-0 |   o.c.i.ServerHandshakeHandler | channelRead: Removing handshake handler from pipeline.
```

收到了配置线程发来的握手信息，然后收到了其发来的```LAYOUT_BOOTSTRAP```的消息，并调用```LayoutServer```的对应处理函数，将最新layout写入本地layout文件，将之前layout写入本地layout历史文件，并更新其所有server的epoch。此处只有LogUnitServer有行为，它发起了一个```SEAL```类型的写操作（其实并没有写日志，此处```batchProcessor```输出信息是为了debug），并输出相应信息。最后返回一个```ACK```消息。

```
          layoutServer-0 |             o.c.i.LayoutServer | handleMessageLayoutBootstrap: Bootstrap with new layout=Layout(layoutServers=[localhost:9000, localhost:9001, localhost:9002, localhost:9003, localhost:9004], sequencers=[localhost:9000, localhost:9001], segments=[Layout.LayoutSegment(replicationMode=CHAIN_REPLICATION, start=0, end=-1, stripes=[Layout.LayoutStripe(logServers=[localhost:9002, localhost:9003]), Layout.LayoutStripe(logServers=[localhost:9004])])], unresponsiveServers=[], epoch=775825, clusterId=null), CorfuMsg(clientID=a6fae62e-2a00-435c-a9a1-13a6131b7526, requestID=0, epoch=775825, buf=PooledSlicedByteBuf(freed), msgType=LAYOUT_BOOTSTRAP, priorityLevel=NORMAL)
LogUnit-BatchProcessor-0 |           o.c.i.BatchProcessor | batchWriteProcessor: updating from 0 to 775825
LogUnit-BatchProcessor-0 |           o.c.i.BatchProcessor | Completed 1 operations
　　　　　　layoutServer-0 |            o.c.i.LogUnitServer | LogUnit sealServerWithEpoch: sealed and flushed with epoch 775825
```

与之同时收到的是，```MANAGEMENT_BOOTSTRAP_REQUEST```的信息，调用```ManagementServer```的对应处理函数。这会更新```serverContext```（这是一个服务器节点上所有server对象共同的）的layout对象，并输出相应信息。最后同样是返回一个```ACK```消息。

```
management-0 |         o.c.i.ManagementServer | Received Bootstrap Layout : Layout(layoutServers=[localhost:9000, localhost:9001, localhost:9002, localhost:9003, localhost:9004], sequencers=[localhost:9000, localhost:9001], segments=[Layout.LayoutSegment(replicationMode=CHAIN_REPLICATION, start=0, end=-1, stripes=[Layout.LayoutStripe(logServers=[localhost:9002, localhost:9003]), Layout.LayoutStripe(logServers=[localhost:9004])])], unresponsiveServers=[], epoch=775825, clusterId=null)
management-0 |            o.c.i.ServerContext | Update to new layout at epoch 775825
```


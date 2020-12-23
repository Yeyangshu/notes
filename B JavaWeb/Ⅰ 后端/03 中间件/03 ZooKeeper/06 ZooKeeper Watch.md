# ZooKeeper Watch

ZooKeeper可以做到统一视图，数据结构是目录树结构

客户端挂掉，另一个怎么知道它挂掉？

- client之间可以做心跳验证

- client2可以watch，watch基于ZooKeeper

方向性、时效性

client1与ZooKeeper之间连接有session，如果session没了（几毫秒），就证明a节点消失了，此时会产生事件，回调（callback）client2

- 两个客户端，一个get数据之后
- client2 watch
- client断开，callback client2
# ZooKeeper API 使用

## 1 同步

```java
public class APP {
    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        // ZooKeeper是有session概念的，没有连接池的概念
        // watch分为两类：new ZooKeeper()时，传入的watch，session级别的，跟path、node没有关系
        // watch的注册只发生在读类型调用，get、exists......
        final CountDownLatch countDownLatch = new CountDownLatch(1);
        final ZooKeeper zooKeeper = new ZooKeeper("127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184", 3000,
                new Watcher() {
                    // watch的回调方法
                    public void process(WatchedEvent event) {
                        Event.KeeperState state = event.getState();
                        Event.EventType type = event.getType();
                        String path = event.getPath();
                        System.out.println("new ZooKeeper watch：" + event.toString());

                        switch (state) {
                            case Unknown:
                                break;
                            case Disconnected:
                                break;
                            case NoSyncConnected:
                                break;
                            case SyncConnected:
                                countDownLatch.countDown();
                                System.out.println("connected");
                                break;
                            case AuthFailed:
                                break;
                            case ConnectedReadOnly:
                                break;
                            case SaslAuthenticated:
                                break;
                            case Expired:
                                break;
                        }

                        switch (type) {
                            case None:
                                break;
                            case NodeCreated:
                                break;
                            case NodeDeleted:
                                break;
                            case NodeDataChanged:
                                break;
                            case NodeChildrenChanged:
                                break;
                        }
                    }
                });

        countDownLatch.await();
        ZooKeeper.States states = zooKeeper.getState();
        switch (states) {
            case CONNECTING:
                System.out.println("CONNECTING...");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("CONNECTED...");
                break;
            case CONNECTEDREADONLY:
                break;
            case CLOSED:
                break;
            case AUTH_FAILED:
                break;
            case NOT_CONNECTED:
                break;
        }

        // 创建目录
        String pathName = zooKeeper.create("/yeyangshu", "old data".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        // 注册监控，获取目录，同时创建watch，如果被观察节点发生变化，异步线程进行回调
        final Stat stat = new Stat();
        byte[] node = zooKeeper.getData("/yeyangshu",
                new Watcher() {
                    public void process(WatchedEvent event) {
                        System.out.println("getData watch：" + event.toString());
                    }
                }, stat);
        System.out.println(new String(node));

        // 修改数据，会触发上面的回调
        Stat stat1 = zooKeeper.setData("/yeyangshu", "new data".getBytes(), 0);
        // 再进行修改，还会触发回调吗？不会，watch是一次性的
        Stat stat2 = zooKeeper.setData("/yeyangshu", "new data01".getBytes(), stat1.getVersion());
    }
}
```

控制台打印

```
new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
connected
CONNECTED...
old data
getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
```

如果想获取完节点信息后继续 watch 节点，有两种情况：

1. 参数为布尔，使用new ZooKeeper()的那个watch， `zooKeeper.getData("/yeyangshu", true, stat);`

   ```java
   final Stat stat = new Stat();
   byte[] node = zooKeeper.getData("/yeyangshu",
                                   new Watcher() {
                                       public void process(WatchedEvent event) {
                                           try {
                                               // true，是default watch，被重新注册，new ZooKeeper()的那个watch
                                               zooKeeper.getData("/yeyangshu", true, stat);
                                           } catch (KeeperException e) {
                                               e.printStackTrace();
                                           } catch (InterruptedException e) {
                                               e.printStackTrace();
                                           }
                                           System.out.println("getData watch：" + event.toString());
                                       }
                                   }, stat);
   ```

   控制台打印：

   ```
   new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
   connected
   CONNECTED...
   old data
   getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
   new ZooKeeper watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
   connected
   ```

2. 参数为 watch，使用刚刚的 watch， `zooKeeper.getData("/yeyangshu", this, stat);`

   ```java
   final Stat stat = new Stat();
   byte[] node = zooKeeper.getData("/yeyangshu",
                                   new Watcher() {
                                       public void process(WatchedEvent event) {
                                           try {
                                               // this，使用刚刚的watch
                                               zooKeeper.getData("/yeyangshu", this, stat);
                                           } catch (KeeperException e) {
                                               e.printStackTrace();
                                           } catch (InterruptedException e) {
                                               e.printStackTrace();
                                           }
                                           System.out.println("getData watch：" + event.toString());
                                       }
                                   }, stat);
   ```

   控制台打印：

   ```
   new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
   connected
   CONNECTED...
   old data
   getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
   getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
   ```

## 2 异步

```java
// 接上面的代码
// 异步调用
System.out.println("async start");
new AsyncCallback.DataCallback() {
    // 回调方法
    @Override
    public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
        System.out.println("async call back");
        System.out.println(ctx.toString());
        System.out.println(new String(data));
    }
}, "abc");
System.out.println("async over");
```

控制台打印

```
new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
connected
CONNECTED...
old data
getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/yeyangshu
async start
async over
async call back
abc
new data01
```




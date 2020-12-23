

pom.xml

```xml
<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
</dependency>

```

第一类

```java
public class ZooKeeperApp {
    public static void main(String[] args) throws IOException, InterruptedException {
        // zk是有session概念的，没有连接池的概念
        // watch（观察、回调）分为两类
        //      第一类：new zk时传入的
        ZooKeeper zk = new ZooKeeper("192.168.163.111:2181,192.168.163.112:2181,192.168.163.113:2181,192.168.163.114:2181",
                3000, new Watcher() {
            // watch的回调方法
            @Override
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                Event.EventType type = event.getType();
                String path = event.getPath();
                System.out.println(event.toString());

                switch (state) {
                    case Unknown:
                        break;
                    case Disconnected:
                        break;
                    case NoSyncConnected:
                        break;
                    case SyncConnected:
                        System.out.println("connected");
                        cd.countDown();
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

        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("ing......");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("ed......");
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

    }
}

控制台：
    ing......
```





```java
public class ZooKeeperApp {
    public static void main(String[] args) throws IOException, InterruptedException {
        // zk是有session概念的，没有连接池的概念
        // watch（观察、回调）分为两类
        //      第一类：new zk时传入的
        final CountDownLatch cd = new CountDownLatch(1);
        ZooKeeper zk = new ZooKeeper("192.168.163.111:2181,192.168.163.112:2181,192.168.163.113:2181,192.168.163.114:2181",
                3000, new Watcher() {
            // watch的回调方法
            @Override
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                Event.EventType type = event.getType();
                String path = event.getPath();
                System.out.println(event.toString());

                switch (state) {
                    case Unknown:
                        break;
                    case Disconnected:
                        break;
                    case NoSyncConnected:
                        break;
                    case SyncConnected:
                        System.out.println("connected");
                        cd.countDown();
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

        cd.await();
        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("ing......");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("ed......");
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

    }
}

控制台：
    WatchedEvent state:SyncConnected type:None path:null
    connected
	ed......
```



```java
public class ZooKeeperApp {
    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        // zk是有session概念的，没有连接池的概念
        // watch（观察、回调）分为两类
        // watch的调用只发生在读类型调用，get、exeits
        //      第一类：new zk时传入的
        final CountDownLatch cd = new CountDownLatch(1);
        ZooKeeper zk = new ZooKeeper("192.168.163.111:2181,192.168.163.112:2181,192.168.163.113:2181,192.168.163.114:2181",
                3000, new Watcher() {
            // watch的回调方法
            @Override
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                Event.EventType type = event.getType();
                String path = event.getPath();
                System.out.println(event.toString());

                switch (state) {
                    case Unknown:
                        break;
                    case Disconnected:
                        break;
                    case NoSyncConnected:
                        break;
                    case SyncConnected:
                        System.out.println("connected");
                        cd.countDown();
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

        cd.await();
        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("ing......");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("ed......");
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

        // 创建节点
        String pathName = zk.create("/ooxx", "olddata".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        Stat stat = new Stat();
        // 取值并注册watch
        byte[] node = zk.getData("/ooxx", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("getData watch：" + event.toString());
            }
        }, stat);
        System.out.println(new String(node));
        // 修改ooxx节点，触发回调
        Stat stat1 = zk.setData("/ooxx", "newdata".getBytes(), 0);
        // 还会触发吗
        Stat stat2 = zk.setData("/ooxx", "newdata".getBytes(), stat1.getVersion());
    }
}
控制台：
    WatchedEvent state:SyncConnected type:None path:null
    connected
    ed......
    olddata
    getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/ooxx
```

3s钟之内可以看到数据，3s之后不见数据

![image-20200803233759002](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200803233759002.png)

怎样再去回调？继续监控

```java
public class ZooKeeperApp {
    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        // zk是有session概念的，没有连接池的概念
        // watch（观察、回调）分为两类
        // watch的调用只发生在读类型调用，get、exeits
        //      第一类：new zk时传入的
        final CountDownLatch cd = new CountDownLatch(1);
        ZooKeeper zk = new ZooKeeper("192.168.163.111:2181,192.168.163.112:2181,192.168.163.113:2181,192.168.163.114:2181",
                3000, new Watcher() {
            // watch的回调方法
            @Override
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                Event.EventType type = event.getType();
                String path = event.getPath();
                System.out.println("new zk watch：" + event.toString());

                switch (state) {
                    case Unknown:
                        break;
                    case Disconnected:
                        break;
                    case NoSyncConnected:
                        break;
                    case SyncConnected:
                        System.out.println("connected");
                        cd.countDown();
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

        cd.await();
        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("ing......");
                break;
            case ASSOCIATING:
                break;
            case CONNECTED:
                System.out.println("ed......");
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

        // 创建节点
        String pathName = zk.create("/ooxx", "olddata".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        Stat stat = new Stat();
        // 取值并注册watch
        byte[] node = zk.getData("/ooxx", new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("getData watch：" + event.toString());
                try {
                    // true default watch 被重新注册 new zk的哪个watch
                    zk.getData("/ooxx", true, stat);
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, stat);
        System.out.println(new String(node));
        // 修改ooxx节点，触发回调
        Stat stat1 = zk.setData("/ooxx", "newdata".getBytes(), 0);
        // 还会触发吗
        Stat stat2 = zk.setData("/ooxx", "newdata".getBytes(), stat1.getVersion());
    }
}
控制台：
        new zk watch：WatchedEvent state:SyncConnected type:None path:null
    connected
    ed......
    olddata
    getData watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/ooxx
    new zk watch：WatchedEvent state:SyncConnected type:NodeDataChanged path:/ooxx
    connected
```


































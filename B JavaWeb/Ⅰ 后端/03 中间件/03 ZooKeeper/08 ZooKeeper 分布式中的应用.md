# ZooKeeper 分布式中的应用

## 1 ZooKeeper 作用

ZooKeeper作用是在分布式环境做分布式协调

- 分布式配置
- 分布式锁
- 等等

## 2 分布式配置

先手动创建测试节点

```shell
[zk: localhost:2181(CONNECTED) 3] create /testConf ""
Created /testConf
[zk: localhost:2181(CONNECTED) 4] ls /
[testConf, zookeeper]
```

**首先模拟情景一，没有节点，新建节点**

启动测试类，线程阻塞：

```
ew ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
-
```

手动创建节点

```
[zk: localhost:2181(CONNECTED) 5] create /testConf/AppConf "olddata"
Created /testConf/AppConf
```

控制台打印

```
olddata
olddata
olddata
olddata
...
```



**模拟情景二，修改节点**

zk-cli

```
[zk: localhost:2181(CONNECTED) 6] set /testConf/AppConf "newdata"
```

控制台打印

```
olddata
newdata
newdata
newdata
newdata
newdata
newdata
newdata
newdata
newdata
...
```

删除节点

```
[zk: localhost:2181(CONNECTED) 7] delete /testConf/AppConf
```

控制台打印日志，然后等待

```
newdata
conf is empty
-
```

重新新建节点

```
[zk: localhost:2181(CONNECTED) 8] create /testConf/AppConf "newdata"
Created /testConf/AppConf
```

控制台打印

```
conf is empty
newdata
newdata
newdata
...
```

## 3 分布式锁

CallbackWatch

```java
package com.yeyangshu.lock;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class CallbackWatch implements Watcher, AsyncCallback.StringCallback, AsyncCallback.Children2Callback {

    ZooKeeper zooKeeper;

    String threadName;

    CountDownLatch countDownLatch = new CountDownLatch(1);

    String pathName;

    /**
     * 加锁
     */
    public void tryLock() {
        try {
            System.out.println(threadName + " create....");
            // 创建完回调StringCallback
            zooKeeper.create("/lock", threadName.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL, this, "abc");
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 释放锁
     */
    public void unLock() {
        try {
            zooKeeper.delete(pathName, -1);
            System.out.println(threadName + " over work....");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void process(WatchedEvent event) {

    }

    /**
     * StringCallback
     *
     * @param rc
     * @param path
     * @param ctx
     * @param name
     */
    @Override
    public void processResult(int rc, String path, Object ctx, String name) {
        if (name != null) {
            System.out.println(threadName + "create node：" + name);
            pathName = name;
            // 监控根目录下的所有子节点，不需要watch，使用this(自己实现Children2Callback)回调
            zooKeeper.getChildren("/", false, this, "sdf");
        }
    }

    /**
     * Children2Callback，走到这里说明已经创建完并且看到自己以及前面的节点
     *
     * @param rc
     * @param path
     * @param ctx
     * @param children
     * @param stat
     */
    @Override
    public void processResult(int rc, String path, Object ctx, List<String> children, Stat stat) {
        System.out.println(threadName + "lock locks......");
//        for (String child : children) {
//            System.out.println(child);
//        }

        // 实际需要对节点排序
        Collections.sort(children);

        // 是不是第一个
        // 是第一个
        // 不是第一个
    }

    public ZooKeeper getZooKeeper() {
        return zooKeeper;
    }

    public void setZooKeeper(ZooKeeper zooKeeper) {
        this.zooKeeper = zooKeeper;
    }

    public String getThreadName() {
        return threadName;
    }

    public void setThreadName(String threadName) {
        this.threadName = threadName;
    }

    public String getPathName() {
        return pathName;
    }

    public void setPathName(String pathName) {
        this.pathName = pathName;
    }
}
```

LockTest

```java
package com.yeyangshu.lock;

import com.yeyangshu.config.ZKUtils;
import org.apache.zookeeper.ZooKeeper;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class LockTest {

    ZooKeeper zooKeeper;

    @Before
    public void conn() {
        zooKeeper = ZKUtils.getZooKeeper();
    }

    @After
    public void close() {
        try {
            zooKeeper.close();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void lock() {
        for (int i = 0; i < 10; i++) {
            new Thread() {
                @Override
                public void run() {
                    // 每一个线程
                    CallbackWatch callbackWatch = new CallbackWatch();
                    callbackWatch.setZooKeeper(zooKeeper);
                    String threadName = Thread.currentThread().getName();
                    callbackWatch.setThreadName(threadName);
                    // 1 抢锁
                    callbackWatch.tryLock();
                    // 2 干活
                    System.out.println("working...");
                    // 3 释放锁
                    callbackWatch.unLock();
                }
            }.start();
        }

        while(true){

        }
    }
}
```

将所有的写完之后，测试能否创建成功，此时打印的内容：

```
new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
Thread-0 create....
Thread-1 create....
Thread-2 create....
Thread-6 create....
Thread-7 create....
Thread-8 create....
Thread-3 create....
Thread-4 create....
Thread-5 create....
Thread-9 create....
Thread-0create node：/lock0000000000
Thread-1create node：/lock0000000001
Thread-2create node：/lock0000000002
Thread-6create node：/lock0000000003
Thread-7create node：/lock0000000004
Thread-8create node：/lock0000000005
Thread-3create node：/lock0000000006
Thread-4create node：/lock0000000007
Thread-5create node：/lock0000000008
Thread-9create node：/lock0000000009
Thread-0lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-1lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-2lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-6lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-7lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-8lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-3lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-4lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-5lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009
Thread-9lock locks......
lock0000000000
lock0000000004
lock0000000003
lock0000000002
lock0000000001
lock0000000008
lock0000000007
lock0000000006
lock0000000005
lock0000000009

```

接下来就要去对所有节点进行排序，筛选

```java
@Override
public void processResult(int rc, String path, Object ctx, List<String> children, Stat stat) {
    // 实际需要对节点排序
    Collections.sort(children);
    // 每个线程判断自己是不是第一个
    int index = children.indexOf(pathName.substring(1));
    if (index == 0) {
        // 是第一个
        System.out.println(threadName + " i am first...");
        try {
            // 防止线程过快
            zooKeeper.setData("/", threadName.getBytes(), -1);
            countDownLatch.countDown();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    } else {
        // 不是第一个，监控前一个，当前一个有变化时做处理，回调StatCallback，WatchedEvent
        zooKeeper.exists("/" + children.get(index - 1), this, this, "sdf");
    }
}
```

测试打印

```java
new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
Thread-0 create....
Thread-9 create....
Thread-6 create....
Thread-3 create....
Thread-2 create....
Thread-1 create....
Thread-5 create....
Thread-4 create....
Thread-7 create....
Thread-8 create....
Thread-1 create node：/lock0000000090
Thread-8 create node：/lock0000000091
Thread-0 create node：/lock0000000092
Thread-6 create node：/lock0000000093
Thread-7 create node：/lock0000000094
Thread-2 create node：/lock0000000095
Thread-4 create node：/lock0000000096
Thread-5 create node：/lock0000000097
Thread-9 create node：/lock0000000098
Thread-3 create node：/lock0000000099
Thread-1 i am first...
Thread-1 working...
Thread-1 over work....
Thread-8 i am first...
Thread-8 working...
Thread-8 over work....
Thread-0 i am first...
Thread-0 working...
Thread-0 over work....
Thread-6 i am first...
Thread-6 working...
Thread-6 over work....
Thread-7 i am first...
Thread-7 working...
Thread-7 over work....
Thread-2 i am first...
Thread-2 working...
Thread-2 over work....
Thread-4 i am first...
Thread-4 working...
Thread-4 over work....
Thread-5 i am first...
Thread-5 working...
Thread-5 over work....
Thread-9 i am first...
Thread-9 working...
Thread-9 over work....
Thread-3 i am first...
Thread-3 working...
Thread-3 over work....
```

有时会出现以下情况，卡住了

```
new ZooKeeper watch：WatchedEvent state:SyncConnected type:None path:null
Thread-0 create....
Thread-9 create....
Thread-6 create....
Thread-3 create....
Thread-2 create....
Thread-1 create....
Thread-5 create....
Thread-4 create....
Thread-7 create....
Thread-8 create....
Thread-1 create node：/lock0000000090
Thread-8 create node：/lock0000000091
Thread-0 create node：/lock0000000092
Thread-6 create node：/lock0000000093
Thread-7 create node：/lock0000000094
Thread-2 create node：/lock0000000095
Thread-4 create node：/lock0000000096
Thread-5 create node：/lock0000000097
Thread-9 create node：/lock0000000098
Thread-3 create node：/lock0000000099
Thread-1 i am first...
Thread-1 working...
Thread-1 over work....
```

解决办法，在线程工作附近睡1秒或Children2Callback中添加如下内容

```java
// 防止线程过快
zooKeeper.setData("/", threadName.getBytes(), -1);
```

可重入锁
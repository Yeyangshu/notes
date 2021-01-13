# Lock

必须在finally块里面释放锁

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 更新对象状态
} finally {
    lock.unlock();
}
```

## 轮询锁与定时锁

```java
// 轮询锁
boolean tryLock();
// 定时锁
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
```

轮询

```java
if (lock.tryLock()) {
    try { 
        // 业务代码
    } finally {
        if (locked) {
            lock.unlock();
        }
    }
}
```

可定时的

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try { 
        // 业务代码
    } finally {
        if (locked) {
            lock.unlock();
        }
    }
}
```

## 可中断的锁获取操作

```java
lock.lockInterruptibly();
try { 
    // 业务代码
} finally {
    if (locked) {
        lock.unlock();
    }
}
```

## 公平性

```java
private static ReentrantLock lock = new ReentrantLock(true);
```


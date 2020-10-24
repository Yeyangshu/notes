

配置管理器 

```java
public class ConfigurationManagerImpl implements InitializingBean {
    private final ReentrantLock lock        = new ReentrantLock();

    /**
     * 保证该初始化顺序。
     */
    private void loadMeta() {
        lock.lock();
        try {
            // 加载各种配置
            loadxxx();
        } catch (Exception e){
            throw e;
        } finally {
            lock.unlock();
        }
    }
}
```




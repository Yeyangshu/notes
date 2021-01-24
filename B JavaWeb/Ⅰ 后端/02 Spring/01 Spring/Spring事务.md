# Spring事务

## 事务管理关键抽象

### TransactionDefinition

#### 事务隔离

- PROPAGATION_REQUIRED

#### 事务传播

#### 事务超时

#### 只读事务

### TransactionStatus

### PlatformTransactionManager

```java
TransactionStatus getTransaction(@Nullable TransactionDefinition definition);
void commit(TransactionStatus status)
void rollback(TransactionStatus status)
```





**默认回滚规则**

默认只把runtime, unchecked exceptions标记为回滚，即RuntimeException及其子类，Error默认也导致回滚。Checked exceptions默认不导致回滚。这些规则和EJB是一样的。

> 当事务运行过程中发生异常时，事务可以被声明为回滚或继续提交。在默认的情况下当发生运行期异常时，事务将被回滚；当发生检查型异常时，既不会馆也不提交，控制权交给外部调用。

**事务与线程**

和JavaEE事务上下文一样，Spring事务和一个线程的执行相关联，底层是一个ThreadLocal<Map<Object, Object>>，就是每个线程一个map，key是DataSource，value是Connection。
# 声明式事务

## 1 Spring事务分类

- 编程式事务：在代码中直接加入处理事务的逻辑，可能需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法

- 声明式事务：在方法的外部添加注解或者直接在配置文件中定义，将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。Spring的AOP恰好可以完成此功能：事务管理代码的固定模式作为一种横切关注点，通过AOP方法模块化，进而实现声明式事务。

## 2 Spring事务配置的属性

-  isolation：设置事务的隔离级别
-  propagation：事务的传播行为
-  noRollbackFor：指定的异常，事务可以不回滚
-  noRollbackForClassName：填写的参数是全类名
-  rollbackFor：指定的异常，事务需要回滚
-  rollbackForClassName：填写的参数是全类名
-  readOnly：设置事务是否为只读事务事务运行期间不允许修改，不允许对数据的修改，否则抛出异常
-  timeout：事务超出指定执行时长后自动终止并回滚，单位是秒

注：抛异常和try/catch不会触发事务回滚

### 2.1 timeout

```java
int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

@Transactional(timeout = 3)
public void checkout(String username, int id){

    bookDao.updateStock(id);
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
}
    
控制台报错：
    org.springframework.transaction.TransactionTimedOutException: Transaction timed out: deadline was Mon Jul 20 20:40:47 CST 2020
```

### 2.2 readOnly

```java
boolean readOnly() default false;

@Transactional(readOnly = true)
public void checkout(String username, int id){
    bookDao.updateStock(id);
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
}
控制台报错：
org.springframework.dao.TransientDataAccessResourceException: PreparedStatementCallback; SQL [update book_stock set stock=stock-1 where id=?]; Connection is read-only. Queries leading to data modification are not allowed; nested exception is java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed
```

### 2.3 noRollbackFor

```java
@Transactional(noRollbackFor = {ArithmeticException.class})
public void checkout(String username, int id){

    bookDao.updateStock(id);
    int i = 1 / 0;
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
}

控制台：
    java.lang.ArithmeticException: / by zero
控制台虽然报错，但是数据（bookDao.updateStock(id);）已经更改，注意报错位置后面的不修改
```

### 2.4 noRollbackForClassName

```java
@Transactional(noRollbackForClassName = {"java.lang.ArithmeticException"})
public void checkout(String username, int id){

    bookDao.updateStock(id);
    int i = 1 / 0;
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
}
```

### 2.5 rollbackFor

```java
@Transactional(rollbackFor = {FileNotFoundException.class})
public void checkout(String username, int id) throws FileNotFoundException {
    bookDao.updateStock(id);
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
    new FileInputStream("aaa.txt");
}
控制台：
    java.io.FileNotFoundException: aaa.txt (系统找不到指定的文件。)   
控制台报错，数据回滚，没有被更改
```

### 2.6 rollbackForClassName

```java
@Transactional(rollbackForClassName = {"java.io.FileNotFoundException"})
public void checkout(String username, int id) throws FileNotFoundException {
    bookDao.updateStock(id);
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username,price);
    new FileInputStream("aaa.txt");
}
```

### 2.7 isolation

与数据库事务一致

```java
Isolation.DEFAULT
Isolation.READ_UNCOMMITTED
Isolation.READ_COMMITTED
Isolation.REPEATABLE_READ
Isolation.SERIALIZABLE
```

### 2.8 propagation

- **PROPAGATION_REQUIRED**：如果不存在外层事务，就主动创建事务；否则使用外层事务
- **PROPAGATION_REQUIRES_NEW**：总是主动开启事务；如果存在外层事务，就将外层事务挂起
- **PROPAGATION_NESTED**：如果不存在外层事务，就主动创建事务；否则创建嵌套的子事务
- **PROPAGATION_MANDATORY**：如果不存在外层事务，就抛出异常；否则使用外层事务
- 
- **PROPAGATION_SUPPORTS**：如果不存在外层事务，就不开启事务；否则使用外层事务
- **PROPAGATION_NOT_SUPPORTED**：总是不开启事务；如果存在外层事务，就将外层事务挂起
- **PROPAGATION_NEVER**：总是不开启事务；如果存在外层事务，则抛出异常

#### 2.8.1 Propagation.REQUIRED（常用）

checkout、updatePrice方法添加REQUIRED属性

```java
@Transactional(propagation = Propagation.REQUIRED)
public void checkout(String username, int id) {
    bookDao.updateStock(id);
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username, price);

}

@Transactional(propagation = Propagation.REQUIRED)
public void updatePrice() {
    bookDao.updatePrice(1);
    //int i = 1 / 0; 若打开此处，抛异常，事务回滚
}
```

mul()方法添加声明式事务

```java
@Transactional
public void mul() {
    bookService.checkout("zhangsan", 1);
    bookService.updatePrice();
}
```

添加测试类

```java
@Test
public void testMul() {
    ApplicationContext context = new ClassPathXmlApplicationContext("jdbcTemplate.xml");
    MultiService multiService = context.getBean("multiService", MultiService.class);
    multiService.mul();
}
```

checkout、updatePrice方法的事务交给mul管理

- checkout、updatePrice方法正常执行，mul结果正常，数据库被修改
- checkout、updatePrice方法发生异常，mul事务回滚，数据库不变

#### 2.8.2 Propagation.REQUIRES_NEW（常用）

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void checkout(String username, int id) {
    bookDao.updateStock(id);
    int price = bookDao.getPrice(id);
    bookDao.updateBalance(username, price);

}

@Transactional(propagation = Propagation.REQUIRED)
public void updatePrice() {
    bookDao.updatePrice(1);
    int i = 1 / 0;
}
```

checkout、updatePrice方法的事务交给mul管理

- checkout方法正常执行，数据库被修改
- updatePrice方法发生异常，事务回滚，数据库不变

#### 2.8.3 Propagation.SUPPORTS

#### 2.8.4 Propagation.NOT_SUPPORTED

#### 2.8.5 Propagation.NEVER

#### 2.8.6 Propagation.MANDATORY

#### 2.8.7 Propagation.NESTED

#### 2.8.8 传播特性总结

1. 事务传播级别是REQUIRED，当checkout()被调用时（假定被另一类中commit()调用），如果checkout()中的代码抛出异常，即便被捕获，commit()中的其他代码都会roll back

2. 事务传播级别是REQUIRES_NEW，如果checkout()中的代码抛出异常，并且被捕获，commit()中的其他代码不会roll back；如果commit()中的其他代码抛出异常，而且没有捕获，不会导致checkout()回滚

3. 事务传播级别是NESTED，如果checkout()中的代码抛出异常，并且被捕获，commit()中的其他代码不会roll back；如果commit()中的其他代码抛出异常，而且没有捕获，会导致checkout()回滚

   **NESTED和NEW区别：**

   PROPAGATION_REQUIRES_NEW 启动一个新的，不依赖于环境的 "内部" 事务。这个事务将被完全 commited 或 rolled back，而不依赖于外部事务，它拥有自己的隔离范围，自己的锁等等。 当内部事务开始执行时，外部事务将被挂起，内务事务结束时，外部事务将继续执行。

   另一方面, PROPAGATION_NESTED 开始一个 "嵌套的" 事务,  它是已经存在事务的一个真正的子事务。嵌套事务开始执行时,  它将取得一个 savepoint。如果这个嵌套事务失败，我们将回滚到此 savepoint。嵌套事务是外部事务的一部分，只有外部事务结束后它才会被提交。

   由此可见，PROPAGATION_REQUIRES_NEW和PROPAGATION_NESTED的最大区别在于, PROPAGATION_REQUIRES_NEW 完全是一个新的事务，而 PROPAGATION_NESTED 则是外部事务的子事务，如果外部事务 commit, 嵌套事务也会被 commit，这个规则同样适用于 roll back（理解，NESTED，外部事务失败了，NESTED会回滚；NEW，外部事务失败了，NEW不会回滚）。



自己写一下文字方案。

面试会问

## 3 xml事务配置

jdbcTemplate.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       https://www.springframework.org/schema/tx/spring-tx.xsd
">
    <context:component-scan base-package="com.mashibing"></context:component-scan>
    <context:property-placeholder location="classpath:dbconfig.properties"></context:property-placeholder>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="driverClassName" value="${jdbc.driverClassName}"></property>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
    </bean>
    <bean id="namedParameterJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
    </bean>
    <!--事务控制-->
    <!--配置事务管理器的bean-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--
    基于xml配置的事务：依赖tx名称空间和aop名称空间
        1、spring中提供事务管理器（切面），配置这个事务管理器
        2、配置出事务方法
        3、告诉spring哪些方法是事务方法（事务切面按照我们的切入点表达式去切入事务方法）
    -->
    <bean id="bookService" class="com.mashibing.service.BookService"></bean>
    <aop:config>
        <aop:pointcut id="txPoint" expression="execution(* com.mashibing.service.*.*(..))"/>
        <!--事务建议：advice-ref:指向事务管理器的配置-->
        <aop:advisor advice-ref="myAdvice" pointcut-ref="txPoint"></aop:advisor>
    </aop:config>
    <tx:advice id="myAdvice" transaction-manager="transactionManager">
        <!--事务属性-->
        <tx:attributes>
            <!--指明哪些方法是事务方法-->
            <tx:method name="*"/>
            <tx:method name="checkout" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"></tx:method>
        </tx:attributes>
    </tx:advice>
</beans>
```
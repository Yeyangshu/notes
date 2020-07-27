## 1 客户端分类

- Jedis
- 

Jedis线程不安全，开启一个Jedis，多线程情况下给谁使用？

解决方案：Jedis连接池



maven

```xml
  <dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
  </dependency>
```



关闭外网访问

```
CONFIG GET *
CONFIG SET protected-mode no
```

## 2 API分类

### 2.1 高级API

封装好的template

- RedisTemplate
- StringRedisTemplate

#### 2.1.1 RedisTemplate

```java
@Component
public class TestRedis {
    @Autowired
    RedisTemplate redisTemplate;

    public void testRedis() {
        redisTemplate.opsForValue().set("hello", "china");
        System.out.println(redisTemplate.opsForValue().get("hello"));
    }
}
```

运行main方法

```java
public static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run(SpringmvcApplication.class, args);
    TestRedis redis = context.getBean(TestRedis.class);
    redis.testRedis();
}
```

Redis数据库显示

```
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x05hello"
```

#### 2.1.2 StringRedisTemplate

```java
@Component
public class TestRedis {
    @Autowired
    StringRedisTemplate redisTemplate;

    public void testRedis() {
        redisTemplate.opsForValue().set("hello", "china");
        System.out.println(redisTemplate.opsForValue().get("hello"));
    }
}
```

运行main方法

```java
public static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run(SpringmvcApplication.class, args);
    TestRedis redis = context.getBean(TestRedis.class);
    redis.testRedis();
}
```

Redis数据库显示

```
1) "hello"
```

### 2.2 低级API

可以使用Redis所有命令

```java
@Component
public class TestRedis {
    @Autowired
    StringRedisTemplate redisTemplate;

    public void testRedis() {
        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        connection.set("hello".getBytes(), "world".getBytes());
        System.out.println(new String(connection.get("hello".getBytes())));
    }
}
```



## 3 Java序列化

https://docs.spring.io/spring-data/redis/docs/2.3.2.RELEASE/reference/html/#redis.hashmappers.root

### 3.1 Jackson2HashMapper

#### 3.1.1 redisTemplate

```java
@Component
public class TestRedis {
    @Autowired
    RedisTemplate stringRedisTemplate;

    @Autowired
    ObjectMapper objectMapper;

    public void testRedis() {
        Person person = new Person();
        person.setId(1);
        person.setName("小明");
        Jackson2HashMapper jm = new Jackson2HashMapper(objectMapper, false);
        redisTemplate.opsForHash().putAll("sean", jm.toHash(person));
        Map map = redisTemplate.opsForHash().entries("sean");
        Person per = objectMapper.convertValue(map, Person.class);
        System.out.println(per.toString());
    }
}
```

控制台：

```json
Person{id=1, name='小明'}
```

Redis数据库

```
"\xac\xed\x00\x05t\x00\x04sean"
```

#### 3.1.2 stringRedisTemplate

使用stringRedisTemplate运行上面的代码，如果value中有非String类型数据，会报错，例如

```
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```

解决方式：使用Jackson序列化器

```java
@Component
public class TestRedis {
    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Autowired
    ObjectMapper objectMapper;

    public void testRedis() {
        Person person = new Person();
        person.setId(1);
        person.setName("小明");
        
        stringRedisTemplate.setHashValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        
        Jackson2HashMapper jm = new Jackson2HashMapper(objectMapper, false);
        stringRedisTemplate.opsForHash().putAll("sean", jm.toHash(person));
        Map map = stringRedisTemplate.opsForHash().entries("sean");
        Person per = objectMapper.convertValue(map, Person.class);
        System.out.println(per.toString());
    }
}
```

运行无报错

问题：如何才能在使用时直接序列化呢？

### 3.2 自定义序列化器

```java
@Configuration
public class MyTemplate {

    @Bean
    public StringRedisTemplate ooxx(RedisConnectionFactory fc){

        StringRedisTemplate tp = new StringRedisTemplate(fc);

        tp.setHashValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        return  tp ;
    }
}
```

测试文件

```java

```


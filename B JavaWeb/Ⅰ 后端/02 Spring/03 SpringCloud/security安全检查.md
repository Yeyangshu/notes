## 安全配置

### 1.1 添加maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 1.2 开启Eureka安全连接

```
spring.security.user.name=yiming
spring.security.user.password=123
```



![image-20200408185532993](C:\Study\InternetArchitect-master\20 架构师三期 SpringCloud微服务架构\images\image-20200408185532993.png)



### 1.3 如果服务注册报错

```
Root name 'timestamp' does not match expected ('instance') for type [simple type, class com.netflix.appinfo.InstanceInfo]
```

是默认开启了防止跨域攻击

#### 1.3.1 手动关闭security跨域攻击

在服务端增加配置类

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// TODO Auto-generated method stub
		http.csrf().disable();
		super.configure(http);
	}

}
```


# Spring Security

## 2.1 简介

官网：https://spring.io/projects/spring-security

## 2.2 密码存储

### 2.1 暴力破解/字典/彩虹表

常见密文存储的几种方式：

- 明文
- hash(明文)
- hash(明文 + 盐)

盐的几种实现：

- 用户名、手机号等，每个账户不一样
- 统一的盐
- 随机盐（保存数据库）
- 随机盐（从密码取）

### 2.2 防止破解

没有绝对安全的网络，即使拿不到密码，也可以发送重放攻击

- 多次加盐取hash

  ![image-20201123141633904](C:\Users\wb-zmx522400\AppData\Roaming\Typora\typora-user-images\image-20201123141633904.png)

- 使用更复杂的单向加密算法比如Bcrypt

- 使用https

- 风控系统

  - 二次安全校验
  - 接口调用安全校验
  - 异地登录等
  - 大额转账

### 2.3 Bcrypt结构

- $是分割符，无意义；
- 2a是bcrypt加密版本号；
- 10是加密次数；
- 22位是salt值；
- 最后的字符串是密码的密文。

#### 2.3.1 Spring Security使用Bcrypt

Spring Security Bcrypt的使用见Spring Security文档 2.3.9

## 2.3 基于内存用户登录

### 2.3.1 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 2.3.2 启动项目

![image-20200508165901345](C:\Study\InternetArchitect-master\20 架构师三期 SpringCloud微服务架构\images\image-20200508165901345.png)

启动成功后会生成一个默认密码

```
Using generated security password: 6e86c6e9-d661-41ae-aabc-bea8817c4f7b
```

接下来访问系统使用用户名 user

UserDetailsServiceAutoConfiguration类

### 2.3.3 自定义用户名密码

#### 2.3.3.1 配置文件

```properties
spring.security.user.name=111
spring.security.user.password=111
```

#### 2.3.3.2 配置类定义

```java
@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {	
        auth.inMemoryAuthentication()
            .withUser("111")
            .password("222")
            .roles("admin");
        //	super.configure(auth);
    }

    // 密码加密
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

#### 2.3.3.3 基于内存Memory存储的多用户

```java
@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {	
        auth.inMemoryAuthentication()
            .withUser("111").password("222").roles("admin")
            .and()
            .withUser("333").password("444").roles("xxoo");
    }
}
```

或者

自定义UserDetailsService

```java
@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public UserDetailsService userDetailsService() {
		// 基于内存的用户存储
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();

        User user = new User("a", new BCryptPasswordEncoder().encode("1"), true, true, true, true, Collections.singletonList(new SimpleGrantedAuthority("xx")));
        manager.createUser(user);
        manager.createUser(User.withUsername("yiming").password(new BCryptPasswordEncoder().encode("xx")).roles("xxz").build());
        return manager;

    }
}
```

### 2.3.3 Security中的User对象

```java
private String password;
private final String username;
private final Set<GrantedAuthority> authorities;
private final boolean accountNonExpired;
private final boolean accountNonLocked;
private final boolean credentialsNonExpired;
private final boolean enabled;
```

### 2.3.4 Session中存储的对象

```
Enumeration<String> attributeNames = request.getSession().getAttributeNames();


//		while (attributeNames.hasMoreElements()) {
//			String string = (String) attributeNames.nextElement();
//			System.out.println(string);
//			System.out.println(request.getSession().getAttribute(string));
//			
//		}

SecurityContext attribute = (SecurityContext)request.getSession().getAttribute("SPRING_SECURITY_CONTEXT");
System.out.println(attribute.getAuthentication().getAuthorities());
```

### 2.3.5 忽略静态请求

```java
@Override
public void configure(WebSecurity web) throws Exception {
    // 不需要登陆就可以访问静态资源
    web.ignoring().antMatchers("/img/**","/js/**");
}
```

### 2.3.6 自定义登录页面

配置类：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // TODO Auto-generated method stub
    http.authorizeRequests()
        //所有请求都需要验证
        .anyRequest().authenticated()
        .and()
        //permitAll 给没登录的 用户可以访问这个地址的权限
        .formLogin().loginPage("/login.html").permitAll();
}
```

页面名字：login.html

```html
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8">
</head>
<body>
登录
<form action="/login" method="post">
    <input name="username" type="text"><br>
    <input name="password" type="password">
    <br>
    <input th:name="${_csrf.parameterName}" th:value="${_csrf.token}">
    <button type="submit">登录</button>
</form>
</body>
</html>
```

也可以设置登录成功到指定页面

```java
.defaultSuccessUrl("/index.html").permitAll()
```

### 2.3.7 自定义表单属性

配类中

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // TODO Auto-generated method stub
    http.authorizeRequests()
        // 所有请求都需要验证
        .anyRequest().authenticated()
        .and()
        // permitAll 给没登录的 用户可以访问这个地址的权限
        .formLogin().loginPage("/login.html")
        // 自定义表单，配置登录页的表单name
        .usernameParameter("xx")
        .passwordParameter("oo")

        .loginProcessingUrl("/login")
        .failureUrl("/login.html?error")
        .defaultSuccessUrl("/").permitAll()
        // 重写HttpSecurity默认开启了csrf，所有post请求都会拦截
        .and()
        .csrf().csrfTokenRepository(new HttpSessionCsrfTokenRepository());
}
```

登录页

```html
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8">
</head>
<body>
登录
<form action="/login" method="post">
    <input name="xx" type="text"><br>
    <input name="oo" type="password">
    <br>
    <input th:name="${_csrf.parameterName}" th:value="${_csrf.token}">
    <button type="submit">登录</button>
</form>
</body>
</html>
```

### 2.3.8 密码加密接口

接口

PasswordEncoder 

三个方法

```java
	/**
	 * Encode the raw password. Generally, a good encoding algorithm applies a SHA-1 or
	 * greater hash combined with an 8-byte or greater randomly generated salt.
	 加密密码
	 */
	String encode(CharSequence rawPassword);

	/**
	 * Verify the encoded password obtained from storage matches the submitted raw
	 * password after it too is encoded. Returns true if the passwords match, false if
	 * they do not. The stored password itself is never decoded.
	 *
	 * @param rawPassword the raw password to encode and match
	 * @param encodedPassword the encoded password from storage to compare with
	 * @return true if the raw password, after encoding, matches the encoded password from
	 * storage
	 校验密码
	 */
	boolean matches(CharSequence rawPassword, String encodedPassword);

	/**
	 * Returns true if the encoded password should be encoded again for better security,
	 * else false. The default implementation always returns false.
	 * @param encodedPassword the encoded password to check
	 * @return true if the encoded password should be encoded again for better security,
	 * else false.
	 
	 是否需要再次加密
	 */
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}
```

实现类

![image-20200511153323518](C:\Study\InternetArchitect-master\20 架构师三期 SpringCloud微服务架构\images\image-20200511153323518.png)

### 2.3.9 密码加密方式

不加密

```java
@Bean
PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```

BCrypt样例

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("111").password(new BCryptPasswordEncoder().encode("111")).roles("admin");
}

@Bean
public BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### 2.3.9 完整的配置样例

```java
package com.yeyangshu.security;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 哪些地址需要登录
        http.authorizeRequests()
            // 所有请求都需要验证
            .anyRequest().authenticated()
            .and()
            // 自定义登录页面
            .formLogin().loginPage("/login.html")
            // 配置登录页的表单name
            .usernameParameter("xx")
            .passwordParameter("oo")
            // permitAll 给没登录的用户可以访问这个地址的权限
            .loginProcessingUrl("/login").permitAll()
            // 登录失败页面
            .failureUrl("/login.html?error")
            // 登录成功页面
            .defaultSuccessUrl("/").permitAll()
            // 登陆失败处理器
            .failureHandler(new AuthenticationFailureHandler() {
                @Override
                public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                    e.printStackTrace();
                    // 判断异常信息 跳转不同页面 比如 密码过期重置
                    httpServletRequest.getRequestDispatcher(httpServletRequest.getRequestURL().toString()).forward(httpServletRequest, httpServletResponse);
                    // 记录登录失败次数 禁止登录
                }
            })
            .and()
            // csrf
            .csrf().csrfTokenRepository(new HttpSessionCsrfTokenRepository());
    }

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## 2.3 基于JDBC用户存储

### 2.3.1 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 2.3.2 配置文件

```properties
spring.datasource.username=root
spring.datasource.password=840416
spring.datasource.url=jdbc:mysql:///mq?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
```

### 2.3.3 建表

Spring Security默认情况下需要两张表，用户表和权限表，可以参考

org.springframework.security.core.userdetails.jdbc.users.ddl

```sql
create table users(username varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not null,enabled boolean not null);
create table authorities (username varchar_ignorecase(50) not null,authority varchar_ignorecase(50) not null,constraint fk_authorities_users foreign key(username) references users(username));
create unique index ix_auth_username on authorities (username,authority);
```

自己测试建表MySQL

```sql
create table users(username varchar(50) not null primary key,password varchar(500) not null,enabled boolean not null);

create table authorities (username varchar(50) not null,authority varchar(50) not null,constraint fk_authorities_users foreign key(username) references users(username));

create unique index ix_auth_username on authorities (username,authority);
```

### 2.3.4 登录实现及用户注册

```java
package com.yeyangshu.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.JdbcUserDetailsManager;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;
import java.io.IOException;

@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 哪些地址需要登录
        http.authorizeRequests()
                // 所有请求都需要验证
                .anyRequest().authenticated()
                .and()
                // 自定义登录页面
                .formLogin().loginPage("/login.html")
                // permitAll 给没登录的用户可以访问这个地址的权限
                .loginProcessingUrl("/login").permitAll()
                // 登录失败页面
                .failureUrl("/login.html?error")
                // 登录成功页面
                .defaultSuccessUrl("/").permitAll()
                // 登陆失败处理器
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        // e.printStackTrace();
                        // 判断异常信息 跳转不同页面 比如 密码过期重置
                        httpServletRequest.getRequestDispatcher(httpServletRequest.getRequestURL().toString()).forward(httpServletRequest, httpServletResponse);
                        // 记录登录失败次数 禁止登录
                    }
                })
                .and()
                // csrf
                .csrf().csrfTokenRepository(new HttpSessionCsrfTokenRepository());
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Autowired
    DataSource dataSource;

    @Bean
    protected UserDetailsService userDetailsService() {

        JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);

        if (manager.userExists("111")) {
            System.out.println("已注册");
        } else {
            manager.createUser(User.withUsername("111")
                    .password(new BCryptPasswordEncoder().encode("111"))
                    .roles("admin")
                    .build()
            );
        }
        return manager;
    }

}
```

或者

```java
package com.yeyangshu.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.JdbcUserDetailsManager;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;
import java.io.IOException;

@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 哪些地址需要登录
        http.authorizeRequests()
                // 所有请求都需要验证
                .anyRequest().authenticated()
                .and()
                // 自定义登录页面
                .formLogin().loginPage("/login.html")
                // permitAll 给没登录的用户可以访问这个地址的权限
                .loginProcessingUrl("/login").permitAll()
                // 登录失败页面
                .failureUrl("/login.html?error")
                // 登录成功页面
                .defaultSuccessUrl("/").permitAll()
                // 登陆失败处理器
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
                        // e.printStackTrace();
                        // 判断异常信息 跳转不同页面 比如 密码过期重置
                        httpServletRequest.getRequestDispatcher(httpServletRequest.getRequestURL().toString()).forward(httpServletRequest, httpServletResponse);
                        // 记录登录失败次数 禁止登录
                    }
                })
                .and()
                // csrf
                .csrf().csrfTokenRepository(new HttpSessionCsrfTokenRepository());
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Autowired
    DataSource dataSource;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        JdbcUserDetailsManager manager = auth.
                jdbcAuthentication()
                .dataSource(dataSource).getUserDetailsService();

        if (manager.userExists("111")) {
            System.out.println("已注册");
        } else {
            // 注册用户
            manager.createUser(User.withUsername("111")
                    .password(new BCryptPasswordEncoder().encode("111"))
                    .roles("admin")
                    .build()
            );
        }
    }
}
```

## 2.4 如何使用mybatis/jpa查询用户

如果不想使用框架自带的UserDetailsManager

### 2.4.1 自定义用户登录查询

新建一个service实现`UserDetailsService`接口

```java
/**
 * 完全自定义
 */
@Service
public class UserService implements UserDetailsService {

    // 查询数据库
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 可以从Redis、MySQL、JPA取
        if (new Random().nextBoolean()) {
            throw new LockedException("用户已锁定");
        } else {
            throw new BadCredentialsException("我错了");
        }
    }
}
```

将service注入到配置

```java
@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    UserService userSrv;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userSrv);
    }
}
```

### 2.4.2 自定义用户权限校验

#### 2.4.2.1 自定义校验器

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class MyAuthprovider implements AuthenticationProvider {

    @Autowired
    UserService userService;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // TODO，防止穷举密码，限制重试次数

        String username = authentication.getPrincipal().toString();
        String rawPassword = authentication.getCredentials().toString();
        // 自定义的方法查询用户
        UserDetails userDetails = userService.loadUserByUsername(username);
        // 密码比对
        // 明文，数据库里存的密文，防程序员偷看
        if (new BCryptPasswordEncoder().matches(rawPassword, userDetails.getPassword())) {
            // 密码通过，登陆成功
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(username, userDetails.getPassword(), userDetails.getAuthorities());
            return authenticationToken;
        } else {
            throw new BadCredentialsException("用户密码错误");
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return true;
    }
}
```

#### 2.4.2.2 配置自定义校验器

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    JdbcUserDetailsManager manager = auth.jdbcAuthentication()
        .dataSource(dataSource).getUserDetailsService();

    auth.authenticationProvider(new MyAuthprovider());
}
```



## 2.5 记住我

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.
        // 哪些地址需要登录
        authorizeRequests()
        //所有请求都需要验证
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .and()
        // 记住我
        .rememberMe()
        .and()
        .csrf().disable()
}
```



![image-20201124095415182](C:\Users\wb-zmx522400\AppData\Roaming\Typora\typora-user-images\image-20201124095415182.png)

## 2.6 同一用户多地点登录

此配置和记住我有冲突

### 2.6.1 踢掉其他已登录的用户

maximumSessions(1)

```java
@Override
protected void configure(HttpSecurity http) throws Exception {		
    http.
        // 哪些地址需要登录
        authorizeRequests()
        //所有请求都需要验证
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .and()
        .csrf().disable()
        .sessionManagement()
        // 允许同时登陆的客户端
        .maximumSessions(1);
}
```

测试结果：

其他浏览器登陆之后，再次刷新页面显示

```
This session has been expired (possibly due to multiple concurrent logins being attempted as the same user).
```

### 2.6.2 禁止其他终端登录

maxSessionsPreventsLogin(true)

```java
@Override
protected void configure(HttpSecurity http) throws Exception {		
    http.
        // 哪些地址需要登录
        authorizeRequests()
        //所有请求都需要验证
        .anyRequest().authenticated()
        .and()
        .formLogin()
        .and()
        .csrf().disable()
        .sessionManagement()
        // 允许同时登陆的客户端
        .maximumSessions(1)
        // 已经有用户登录之后，不允许相同用户再次登录
        .maxSessionsPreventsLogin(true);
}
```



### 2.6.3 及时清理过期session

```java
@Configuration
@EnableWebSecurity
public class MyConfig extends WebSecurityConfigurerAdapter {
    @Bean
    HttpSessionEventPublisher httpSessionEventPublisher() {
        return new HttpSessionEventPublisher();
    }
}
```

## 防火墙

### ip白名单

#### 指定ip可以不登录

```
		http.
		// 哪些 地址需要登录
		authorizeRequests()
		//所有请求都需要验证
		.anyRequest().authenticated()
		
		.antMatchers("/ip1").hasIpAddress("127.0.0.1")
```

#### 禁止ip访问

用Filter 实现、或者用HandlerInterceptor 实现

### StrictHttpFirewall

spring security 默认使用StrictHttpFirewall限制用户请求

#### method

缺省被允许的`HTTP method`有 [`DELETE`, `GET`, `HEAD`, `OPTIONS`, `PATCH`, `POST`, `PUT`]

#### URI

**在其`requestURI`/`contextPath`/`servletPath`/`pathInfo`中，必须不能包含以下字符串序列之一 :**

```
["//","./","/…/","/."]
```



#### 分号

```
;或者%3b或者%3B
// 禁用规则
setAllowSemicolon(boolean)
```



#### 斜杠

```
%2f`或者`%2F
// 禁用规则
setAllowUrlEncodedSlash(boolean)
```

#### 反斜杠

```
\或者%5c或者%5B
// 禁用规则
setAllowBackSlash(boolean)
```

#### 英文句号

```
%2e或者%2E
// 禁用规则
setAllowUrlEncodedPeriod(boolean)
```



#### 百分号

```
%25
// 禁用规则
setAllowUrlEncodedPercent(boolean)
```

#### 防火墙与sql注入

' ; -- % 多数非法字符已经在请求的参数上被禁用

为啥用户名不能有特殊字符

preparestatement 

awf前端拦截

## 自定义配置

### 指定登录的action

```
.loginProcessingUrl("/login")
```

### 指定登录成功后的页面

			//直接访问登录页面时返回的地址,如果访问的是登录页的话返回指定的地址
			.defaultSuccessUrl("/",true)
			 //必须返回指定地址
			.defaultSuccessUrl("/",true)

### 指定错误页

		//指定错误页
		.failureUrl("/error.html?error1")

### 注销登录

#### 默认方式

```
<a href="/logout">GET logout</a>
<br />
<form action="/logout" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <input type="submit" value="POST Logout"/>
</form>
```

#### 自定义url

```
		.and()
		.logout()
		.logoutUrl("/out")
```

### 增加退出处理器

```
		.addLogoutHandler(new LogoutHandler() {
			
			@Override
			public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
				// TODO Auto-generated method stub
				System.out.println("退出1");
			}
		})
		
		
		.addLogoutHandler(new LogoutHandler() {
			
			@Override
			public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
				// TODO Auto-generated method stub
				System.out.println("退出2");
			}
		})
```

### 登录成功处理器

不同角色 跳转到不同页面

		.successHandler(new AuthenticationSuccessHandler() {
			
			@Override
			public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
					Authentication authentication) throws IOException, ServletException {
				// TODO Auto-generated method stub
				
				System.out.println("登录成功1");
				// 根据权限不同，跳转到不同页面
				request.getSession().getAttribute(name)
				request.getRequestDispatcher("").forward(request, response);
			}
		})

其中 Authentication 参数包含了 用户权限信息

### 登录失败处理器

```
		.failureHandler(new AuthenticationFailureHandler() {
			
			@Override
			public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
					AuthenticationException exception) throws IOException, ServletException {
				// TODO Auto-generated method stub
				exception.printStackTrace();
				request.getRequestDispatcher(request.getRequestURL().toString()).forward(request, response);
			}
		})
```

可以限制登录错误次数

#### 常见登录异常

![image-20200511181553196](C:\Study\InternetArchitect-master\20 架构师三期 SpringCloud微服务架构\images\image-20200511181553196.png)

**LockedException** 账户被锁定

**CredentialsExpiredException** 密码过期

**AccountExpiredException** 账户过期

**DisabledException** 账户被禁用

**BadCredentialsException** 密码错误

**UsernameNotFoundException** 用户名错误

## 访问权限

访问权限可以配置URL匹配用户角色或权限

		http.authorizeRequests()
		.antMatchers("/admin/**").hasRole("admin")
		.antMatchers("/user/**").hasRole("user")
	@Bean

### Ant 风格路径表达式

| 通配符 | 说明                    |
| ------ | ----------------------- |
| ?      | 匹配任何单字符          |
| *      | 匹配0或者任意数量的字符 |
| **     | 匹配0或者更多的目录     |

#### 例子

| URL路径            | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| /app/*.x           | 匹配(Matches)所有在app路径下的.x文件                         |
| /app/p?ttern       | 匹配(Matches) /app/pattern 和 /app/pXttern,但是不包括/app/pttern |
| /**/example        | 匹配(Matches) /app/example, /app/foo/example, 和 /example    |
| /app/**/dir/file.* | 匹配(Matches) /app/dir/file.jsp, /app/foo/dir/file.html,/app/foo/bar/dir/file.pdf, 和 /app/dir/file.java |
| /**/*.jsp          | 匹配(Matches)任何的.jsp 文件                                 |



#### 最长匹配原则

最长匹配原则(has more characters)
说明，URL请求/app/dir/file.jsp，现在存在两个路径匹配模式/**/*.jsp和/app/dir/*.jsp，那么会根据模式/app/dir/*.jsp来匹配

#### 匹配顺序

security像shiro一样，权限匹配有顺序，比如不能把.anyRequest().authenticated()写在其他规则前面

### 权限继承

​	

	RoleHierarchy roleHierarchy() {
		
		RoleHierarchyImpl impl = new RoleHierarchyImpl();
		impl.setHierarchy("ROLE_admin > ROLE_user");
		
		return impl;
		
	}



## 权限控制细粒度注解

## 角色匹配

### 配置类

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
```

### **securedEnabled = true** 

方法验证

```
	@GetMapping("/hi_admin")
	@Secured({"ROLE_admin","ROLE_user"})
	public Authentication hi() {
```

开启简单验证，之验证单一角色是否持有



### **prePostEnabled = true** 

支持更复杂的角色匹配，比如必须同时包含两个角色

	**需包含user角色**

```
	@PreAuthorize("hasRole('ROLE_user')")

```

**需包含user或admin角色**

```
	@PreAuthorize("haAnyRole('ROLE_admin','ROLE_user')")
```



**同时需包含user和admin角色**

```
	@PreAuthorize("hasRole('ROLE_admin') AND hasRole('ROLE_user')")
```



### 根据方法返回值判断是否有权限

```
	@PostAuthorize("returnObject==1")
```





### 方法拦截

```
	@GetMapping("/hi")
	@PreAuthorize("hasRole('ROLE_admin')")
	public String hi() {
	//	UserDetailsServiceAutoConfiguration
		System.out.println("来啦老弟~！");
		return "hi";
	}
	
	
	@PreAuthorize("hasRole('ROLE_user')")
	@GetMapping("/hiUser")
	public String hiuser() {
	//	UserDetailsServiceAutoConfiguration
		System.out.println("来啦老弟~！");
		return "hi";
	}
```

### 获取用户权限信息和UserDetails

```
	Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
	
		authentication.getPrincipal()
```



## 图形验证码

目的：防机器暴力登陆

### Kaptcha 

| Constant                         | 描述                                                         | 默认值                                                |
| -------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| kaptcha.border                   | 图片边框，合法值：yes , no                                   | yes                                                   |
| kaptcha.border.color             | 边框颜色，合法值： r,g,b (and optional alpha) 或者 white,black,blue. | black                                                 |
| kaptcha.image.width              | 图片宽                                                       | 200                                                   |
| kaptcha.image.height             | 图片高                                                       | 50                                                    |
| kaptcha.producer.impl            | 图片实现类                                                   | com.google.code.kaptcha.impl.DefaultKaptcha           |
| kaptcha.textproducer.impl        | 文本实现类                                                   | com.google.code.kaptcha.text.impl.DefaultTextCreator  |
| kaptcha.textproducer.char.string | 文本集合，验证码值从此集合中获取                             | abcde2345678gfynmnpwx                                 |
| kaptcha.textproducer.char.length | 验证码长度                                                   | 5                                                     |
| kaptcha.textproducer.font.names  | 字体                                                         | Arial, Courier                                        |
| kaptcha.textproducer.font.size   | 字体大小                                                     | 40px.                                                 |
| kaptcha.textproducer.font.color  | 字体颜色，合法值： r,g,b  或者 white,black,blue.             | black                                                 |
| kaptcha.textproducer.char.space  | 文字间隔                                                     | 2                                                     |
| kaptcha.noise.impl               | 干扰实现类                                                   | com.google.code.kaptcha.impl.DefaultNoise             |
| kaptcha.noise.color              | 干扰 颜色，合法值： r,g,b 或者 white,black,blue.             | black                                                 |
| kaptcha.obscurificator.impl      | 图片样式：<br />水纹 com.google.code.kaptcha.impl.WaterRipple <br /> 鱼眼 com.google.code.kaptcha.impl.FishEyeGimpy <br /> 阴影 com.google.code.kaptcha.impl.ShadowGimpy | com.google.code.kaptcha.impl.WaterRipple              |
| kaptcha.background.impl          | 背景实现类                                                   | com.google.code.kaptcha.impl.DefaultBackground        |
| kaptcha.background.clear.from    | 背景颜色渐变，开始颜色                                       | light grey                                            |
| kaptcha.background.clear.to      | 背景颜色渐变， 结束颜色                                      | white                                                 |
| kaptcha.word.impl                | 文字渲染器                                                   | com.google.code.kaptcha.text.impl.DefaultWordRenderer |
| kaptcha.session.key              | session key                                                  | KAPTCHA_SESSION_KEY                                   |
| kaptcha.session.date             | session date                                                 | KAPTCHA_SESSION_DATE                                  |



作者：撸帝
链接：https://www.jianshu.com/p/a3525990cd82
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```
<!-- https://mvnrepository.com/artifact/com.github.penggle/kaptcha -->
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>

```

### 添加一个前置Filter

```
	http.addFilterBefore(new CodeFilter(), UsernamePasswordAuthenticationFilter.class);
```



```java
public class CodeFilter implements Filter {

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest req = (HttpServletRequest)request;
		HttpServletResponse resp = (HttpServletResponse)response;
		
		String uri = req.getServletPath();


		if(uri.equals("/login") && req.getMethod().equalsIgnoreCase("post")) {

		
			String sessionCode = req.getSession().getAttribute(Constants.KAPTCHA_SESSION_KEY).toString();
			String formCode = req.getParameter("code").trim();
				
			if(StringUtils.isEmpty(formCode)) {
				throw new RuntimeException("验证码不能为空");
			}
			if(sessionCode.equalsIgnoreCase(formCode)) {
				
				System.out.println("验证通过");
				
			}		
					System.out.println(req.getSession().getAttribute(Constants.KAPTCHA_SESSION_KEY));
			throw new AuthenticationServiceException("xx");
		}

		chain.doFilter(request, response);
		
	}

```

显示验证码的Controller

```
	@GetMapping("/kaptcha")
    public void getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpSession session = request.getSession();
        response.setDateHeader("Expires", 0);
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
        response.setHeader("Pragma", "no-cache");
        response.setContentType("image/jpeg");
        String capText = captchaProducer.createText();
        
        
        session.setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);
        BufferedImage bi = captchaProducer.createImage(capText);
        ServletOutputStream out = response.getOutputStream();
        ImageIO.write(bi, "jpg", out);
        try {
            out.flush();
        } finally {
            out.close();
        }
    }
```



### 配置类

```
package com.mashibing.admin;

import java.util.Properties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.google.code.kaptcha.impl.DefaultKaptcha;
import com.google.code.kaptcha.util.Config;

@Configuration
public class Kaconfig {
    @Bean
    public DefaultKaptcha getDefaultKaptcha(){
        DefaultKaptcha captchaProducer = new DefaultKaptcha();
        Properties properties = new Properties();
        properties.setProperty("kaptcha.border", "yes");
        properties.setProperty("kaptcha.border.color", "105,179,90");
        properties.setProperty("kaptcha.textproducer.font.color", "blue");
        properties.setProperty("kaptcha.image.width", "310");
        properties.setProperty("kaptcha.image.height", "240");
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        properties.setProperty("kaptcha.session.key", "code");
        properties.setProperty("kaptcha.textproducer.char.length", "4");
    //    properties.setProperty("kaptcha.textproducer.char.string", "678");
        properties.setProperty("kaptcha.obscurificator.impl", "com.google.code.kaptcha.impl.ShadowGimpy");
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
        Config config = new Config(properties);
        captchaProducer.setConfig(config);
        return captchaProducer;

    }
}

```



### 短信验证码

### 人机交互验证

## 集群化服务之Session共享

### SpringSession + Redis

#### 配置文件

```
spring.redis.host=localhost
#spring.redis.password=
spring.redis.port=6379

spring.security.user.name=123
spring.security.user.password=123

server.port=81
```

#### 依赖

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

``
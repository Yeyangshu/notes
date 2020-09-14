下载dubbo-admin

https://github.com/apache/dubbo-admin/blob/develop/README_ZH.md



老项目拆分微服务，git在原来的基础上checkout一个新的分支

包

- dubbo-oa-consumer
- dubbo-oa-provider

# 1 dubbo-oa-provider

有entity、service、mapper

## 1.1 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.yeyangshu</groupId>
	<artifactId>oa</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>oa office provider</description>

	<properties>
		<java.version>1.8</java.version>
		<dubbo.version>2.7.7</dubbo.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.0.1</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>

		<!--  分页插件 https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.12</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.11</version>
		</dependency>

		<!--Dubbo-->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>4.2.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.2.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.14</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



## 1.1 抽象service

1. IAccountService

   ```java
   package com.yeyangshu.service;
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.RespStat;
   import com.yeyangshu.entity.Account;
   
   import java.util.List;
   
   public interface IAccountService {
       Account findByLoginNameAndPassword(String loginName, String password);
   
       List<Account> findAll();
   
       PageInfo<Account> findByPage(int pageNum, int pageSize);
   
       RespStat deleteById(int id);
   
       void update(Account account);
   }
   ```

   AccountServiceImpl

   ```java
   package com.yeyangshu.service;
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.RespStat;
   import com.yeyangshu.entity.Account;
   import com.yeyangshu.mapper.AccountExample;
   import com.yeyangshu.mapper.AccountMapper;
   import org.apache.dubbo.config.annotation.Service;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;
   
   import java.util.List;
   
   @Component
   @Service(version = "1.0.0", timeout = 10000, interfaceClass = IAccountService.class)
   public class AccountServiceImpl implements IAccountService {
   
       @Autowired
       AccountMapper accountMapper;
   
       /**
        * 通过用户名和密码取出用户的信息
        * @param loginName
        * @param password
        * @return
        */
       @Override
       public Account findByLoginNameAndPassword(String loginName, String password) {
           return accountMapper.findByLoginNameAndPassword(loginName,password);
       }
   
       /**
        * 所有用户信息
        * @return
        */
       @Override
       public List<Account> findAll() {
           AccountExample accountExample = new AccountExample();
           return accountMapper.selectByExample(accountExample);
       }
   
       /**
        * 所有用户信息分页功能
        * @param pageNum
        * @param pageSize
        * @return
        */
       @Override
       public PageInfo<Account> findByPage(int pageNum, int pageSize) {
           List<Account> accounts = accountMapper.selectByPermission();
           System.out.println(accounts.toString());
   
           AccountExample example = new AccountExample();
           List<Account> accountList = accountMapper.selectByExample(example);
           // 导航页码数固定位5页
           return new PageInfo<>(accountList, 5);
       }
   
       /**
        * 操作数据库进行删除操作
        * 注意事项：
        * 1、需要提醒用户是否真的删除
        * 2、使用删除标记，使数据不会永远删除（同理 update，只增加而不去直接修改表内容）
        * @param id
        * @return
        */
       @Override
       public RespStat deleteById(int id) {
           // 删除后会返回行数
           int rows = accountMapper.deleteByPrimaryKey(id);
           System.out.println("delete by id, rows is ：" + rows);
           if (rows == 1) {
               return RespStat.build(200);
           } else {
               return RespStat.build(500, "删除错误");
           }
       }
   
       /**
        * 更新用户信息
        * @param account
        */
       @Override
       public void update(Account account) {
           accountMapper.updateByPrimaryKeySelective(account);
       }
   }
   
   ```

   

2. IPermissionService

   ```java
   package com.yeyangshu.service;
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.entity.Permission;
   
   import java.util.List;
   
   public interface IPermissionService {
       PageInfo<Permission> findByPage(int pageNum, int pageSize);
   
       Permission findById(int id);
   
       List<Permission> findAll();
   
       void update(Permission permission);
   
       void add(Permission permission);
   }
   
   ```

   PermissionServiceImpl

   ```java
   package com.yeyangshu.service;
   
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.entity.Permission;
   import com.yeyangshu.mapper.PermissionExample;
   import com.yeyangshu.mapper.PermissionMapper;
   import org.apache.dubbo.config.annotation.Service;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;
   
   import java.util.List;
   
   /**
    * 权限管理
    * @author yeyangshu
    * @version 1.0
    * @date 2020/8/30 11:14
    */
   @Component
   @Service(version = "1.0.0", timeout = 10000, interfaceClass = IPermissionService.class)
   public class PermissionServiceImpl implements IPermissionService {
       @Autowired
       PermissionMapper permissionMapper;
   
       /**
        * 分页查询
        * @param pageNum
        * @param pageSize
        * @return
        */
       @Override
       public PageInfo<Permission> findByPage(int pageNum, int pageSize) {
           PermissionExample example = new PermissionExample();
           List<Permission> permissionList = permissionMapper.selectByExample(example);
           // 导航页码数固定位5页
           return new PageInfo<>(permissionList, 5);
       }
   
       /**
        * 通过主键 id 查找权限信息
        * @param id
        * @return
        */
       @Override
       public Permission findById(int id) {
           Permission permission = permissionMapper.selectByPrimaryKey(id);
           return permission;
       }
   
       /**
        * 查找所有权限
        * @return
        */
       @Override
       public List<Permission> findAll() {
           PermissionExample permissionExample = new PermissionExample();
           return permissionMapper.selectByExample(permissionExample);
       }
   
       @Override
       public void update(Permission permission) {
           // 有值的进行更新
           permissionMapper.updateByPrimaryKeySelective(permission);
       }
   
       @Override
       public void add(Permission permission) {
           permissionMapper.insertSelective(permission);
       }
   }
   
   ```

3. IRoleService

   ```java
   package com.yeyangshu.service;
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.entity.Role;
   
   public interface IRoleService {
       PageInfo<Role> findByPage(int pageNum, int pageSize);
   
       Role findById(int id);
   
       void addPermission(int id, int[] permissionIds);
   }
   
   ```

   RoleServiceImpl

   ```java
   package com.yeyangshu.service;
   
   import com.github.pagehelper.PageInfo;
   import com.yeyangshu.entity.Permission;
   import com.yeyangshu.entity.Role;
   import com.yeyangshu.mapper.RoleExample;
   import com.yeyangshu.mapper.RoleMapper;
   import org.apache.dubbo.config.annotation.Service;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;
   
   import java.util.List;
   
   /**
    * 角色管理
    * @author yeyangshu
    * @version 1.0
    * @date 2020/8/30 11:15
    */
   @Component
   @Service(version = "1.0.0", timeout = 10000, interfaceClass = IRoleService.class)
   public class RoleServiceImpl implements IRoleService {
       @Autowired
       RoleMapper roleMapper;
   
       @Override
       public PageInfo<Role> findByPage(int pageNum, int pageSize) {
           RoleExample example = new RoleExample();
           List<Role> roleList = roleMapper.selectByExample(example);
           // 导航页码数固定位5页
           return new PageInfo<>(roleList, 5);
       }
   
       /**
        * 查找角色信息
        * @param id
        * @return
        */
       @Override
       public Role findById(int id) {
           return roleMapper.findById(id);
       }
   
       /**
        * 批量为角色添加权限
        * @param id
        * @param permissionIds
        */
       @Override
       public void addPermission(int id, int[] permissionIds) {
           roleMapper.addPermission(id, permissionIds);
       }
   }
   
   ```



## 2 entity/mapper直接拷贝

## 3 启动类

```java
@SpringBootApplication
@MapperScan(value = "com.yeyangshu.mapper")
public class DubboOaProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboOaProviderApplication.class, args);
    }

}
```

## 4 properties

```proper
# 自动补全实体类的包名称
mybatis.type-aliases-package=com.yeyangshu.entity
# 对应的sql映射
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml

#禁用thymeleaf缓存
spring.thymeleaf.cache: false

spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,file:d:/study/project/springboot-oa/uploads

#MyBatis显示SQL
logging.level.com.yeyangshu.mapper=debug

#properties使用 Unicode 编码
systemConfig.systemName=\u4e2d\u56fd\u77f3\u5316

#dubbo
#服务端口号
server.port=8085
spring.application.name=oa-provider

dubbo.scan.base-packages=com.yeyangshu.service
dubbo.protocol.name=dubbo
dubbo.protocol.port=666
dubbo.protocol.host=127.0.0.1
#dubbo注册中心
dubbo.registry.address=zookeeper://127.0.0.1:2181

加一下数据库的
```

# 2 dubbo-oa-consumer

```
<groupId>com.yeyangshu</groupId>
<artifactId>oa-provider</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>oa</name>
<description>OA office</description>
```

有entity、controller、filter、service，前端相关东西

### 2.1 controller

```java
所有的@Service改成
    
@Reference(version = "1.0.0")
IAccountService accountService;
```

### 2.2 service

只复制接口

### 2.3 启动类

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class DubboOaConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(DubboOaConsumerApplication.class, args);
	}

}
```

### 2.4 properties

```properti
systemConfig.systemName=\u4e2d\u56fd\u77f3\u5316

server.port=8082
spring.application.name=DemoConsumer
dubbo.scan.base-packages=com.yeyangshu.service
#dubbo注册中心
dubbo.registry.address=zookeeper://127.0.0.1:2181


```

### 2.5 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.yeyangshu</groupId>
	<artifactId>oa</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>dubbo-oa-consumer</name>
	<description>oa office consumer</description>

	<properties>
		<java.version>1.8</java.version>
		<dubbo.version>2.7.7</dubbo.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.0.1</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>

		<!--  分页插件 https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.12</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.11</version>
		</dependency>

		<!--Dubbo-->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo</artifactId>
			<version>${dubbo.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>4.2.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.2.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.14</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

# 3 注册微信开发账号

git在原来的再基础上checkout一个新的分支

微信公众平台：

网址：https://mp.weixin.qq.com/cgi-bin/registermidpage?action=index&lang=zh_CN&token=

https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Requesting_an_API_Test_Account.html



小插件：natapp，搭公网，可以绑定私服域名，www.yeyangshu.com -> 127.0.0.1:8080

配置域名。



pom

添加微信的依赖

```xml
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>


        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.9</version>
        </dependency>


        <dependency>
            <groupId>com.github.liyiorg</groupId>
            <artifactId>weixin-popular</artifactId>
            <version>2.8.28</version>
        </dependency>

        <!-- dubbo -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
        </dependency>


        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.2.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
        </dependency>
```
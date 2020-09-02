# SpringBoot

- MVC架构思想
- 使用STS构建SpringBoot项目
- 使用SpringBoot构建MVCWeb项目
- MVCWeb项目中的注入
- 热部署

## 1 介绍

SpringBoot主要解决的是在微服务的架构下简化配置（有快速配置）、前后端分离、快速开发

优点：

- 提供了快速启动入门
- 开箱即用、提供默认配置
- 内嵌容器化web项目
- 没有冗余代码生成和xml配置要求

## 2 使用步骤

### 2.1 创建项目

创建SpringBoot项目的几种方式：

- 官网的Initializr
- 使用Eclipse、STS、Idea等IDE创建Maven项目并引入依赖
- 使用STS插件的Spring Initializr创建项目

访问http://start.spring.io/  进入Spring项目Initializr，生成下载demo.zip

### 2.2 导入项目

Import一个Maven项目。。。。

### 2.3 启动项目

- 直接run启动程序里的Main()方法
- 安装过STS插件或使用STS可以在项目上右键RunAS -> Spring Boot APP

## 3 个性化设置

在application.properties设置

### 3.1 修改启动banner

在resources目录下新建banner.txt

- 英文：http://www.network-science.de/ascii/ 
- 图片：https://www.degraeve.com/img2txt.php

## 4 SpringBoot+Thymeleaf模板

### 4.1 配置文件

application.properties：把所有的配置全放在这个文件里，方便统一管理，maven也可以做到

#### 4.1.1 修改Tomcat端口

```
server.port=90
```

#### 4.1.2 修改项目路径

```
server.servlet.context-path=/demo 
```

### 4.2 简单应用

#### 4.2.1 简单的restful api应用

pom.xml，thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Controller代码

```java
/**
 * 在我们访问  http://主机名：端口号/context-path/Controller的URI/方法的URI
 * http://localhost:80/boot/user/list
 * @author Administrator
 * @Controller 加入Spring容器管理，单例
 */
@Controller
public class MainController {

	/**
	 * String 类型的返回值，会去resources/templates找模板文件
	 * @return
	 */
	@RequestMapping("/list")
	public String list(ModelMap map) {
		
		return "list";
	}
}
```

模板文件代码，list.html，位置在resources/templates/list.html

```html
<html>

<!-- 占位符 -->
<h1>Hello,World!</h1>

</html>
```

#### 4.2.2 复杂的restful api应用

整体项目结构

-java

	-controller
	
		-**MainController**
	
	-service
	
		-**CityService**
	
	-dao
	
		-**CityDAO**
	
	-domain
	
		-**City**
	
	-启动类

-resources

	-templates
	
		-**add.html**
	
		-**list.html**



1. MainController

   ```java
   /**
    * City服务
    */
   @Controller
   public class MainController {
   
   	@Autowired
   	CityService citySrv;
   
   	/**
   	 * 城市添加信息页面
   	 * @return add.html
   	 */
   	@RequestMapping("/addPage")
   	public String addPage() {
   		return "add";
   	}
   
   	/**
   	 * 城市添加接口
   	 * @param city
   	 * @param map
   	 * @return
   	 */
   	@RequestMapping("/add")
   	public String add(@ModelAttribute City city,Model map) {
   		System.out.println(city.toString());
   		String success =citySrv.add(city);
   		map.addAttribute("success", success);
   		return "add";
   	}
   
   	/**
   	 * 获取全部城市列表接口
   	 * @param map
   	 * @return
   	 */
   	@RequestMapping("/list")
   	public String list(Model map) {
   		List<City> list = citySrv.findAll();
   		map.addAttribute("list", list);
   		return "list";
   	}
   }
   ```

2. CityService

   ```java
   @Service
   public class CityService {
   
   	@Autowired
   	CityDAO cityDao;
   	
   	public List<City> findAll() {
   		return cityDao.findAll();
   	}
   
   
   	public String add(Integer id, String name) {
   		City city = new City();
   		city.setId(id);
   		city.setName(name);
   		try {
   			cityDao.save(city);
   			return "保存成功";
   		} catch (Exception e) {
   			return "保存失败";
   		}
   		
   	}
   	
   	public String add(City city) {
   		try {
   			cityDao.save(city);
   			return "保存成功";
   		} catch (Exception e) {
   			return "保存失败";
   		}
   	}
   }
   ```

3. CityDao

   ```java
   @Repository
   public class CityDAO {
   
       /**
        * 在内存中虚拟出一份数据，需要保证线程安全
        */
       static Map<Integer, City> dataMap = Collections.synchronizedMap(new HashMap<Integer, City>());
   
       public List<City> findAll() {
           return new ArrayList<>(dataMap.values());
       }
   
       public void save(City city) throws Exception {
           City data = dataMap.get(city.getId());
           if (data != null) {
               throw new Exception("数据已存在");
           } else {
               dataMap.put(city.getId(), city);
               System.out.println("数据已添加");
           }
       }
   }
   ```

4. City

   ```java
   public class City {
   
       private Integer id;
       private String name;
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "City{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   '}';
       }
   }
   ```

5. add.html

   ```html
   <html xmlns:th="http://www.w3.org/1999/xhtml">
   City ADD
   <hr>
   <h1 th:text="${success}"></h1>
   <hr>
   
   <form action="add" method="post">
       id:<input name="id" type="text">
       <br/>
       name:<input name="name" type="text">
       <br/>
       <input type="submit" value="submit">
   </form>
   
   </html>
   ```

6. list.html

   ```html
   <html xmlns:th="http://www.w3.org/1999/xhtml">
   <body>
   City List
   <hr>
   <table border="1">
       <tr>
           <th>ID</th>
           <th>Name</th>
       </tr>
   
       <tr th:each="city : ${list}">
           <!-- EL JSTL-->
           <td th:text="${city.id}"></td>
           <td th:text="${city.name}"></td>
       </tr>
   
   </table>
   </body>
   </html>
   ```

## 5 Spring Data JPA+MySQL数据库

### 5.1 JPA简单项目

pom.xml

```xml
<!--JPA-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--MySQL-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

1. controller

   ```java
   @Controller
   @RequestMapping("/city")
   public class MainController {
   
       @Autowired
       CityService citySrv;
   
       @RequestMapping("/list")
       public String list(Model map) {
   
           List<City> list = citySrv.findAll();
   
           map.addAttribute("list", list);
           return "list";
       }
   
   }
   ```

2. service

   ```java
   @Service
   public class CityService {
   
   	@Autowired
   	// Repository == Dao
   	CityRepository cityRepo;
   	
   	public List<City> findAll() {
   		List<City> findAll = cityRepo.findAll();
   		return findAll;
   	}
   
   }
   ```

3. repository、直接继承JpaRepository

   ```java
   /**
    * 继承JpaRepository<T, ID>
    * T：实体类
    * ID：实体类id
    */
   public interface CityRepository extends JpaRepository<City, Integer> {
   
   }
   ```

4. entity

   ```java
   @Entity
   @Table(name = "city")
   public class City {
   
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
       private String name;
   
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
   }
   ```

5. list.html

   ```html
   <html xmlns:th="http://www.w3.org/1999/xhtml">
   <body>
   City List
   <hr>
   <table border="1">
       <tr>
           <th>ID</th>
           <th>Name</th>
       </tr>
   
       <tr th:each="city : ${list}">
           <!-- EL JSTL-->
           <td th:text="${city.id}"></td>
           <td th:text="${city.name}"></td>
       </tr>
   </table>
   </body>
   </html>
   ```

6. application.properties

   ```properties
   server.port=80
   spring.datasource.url=jdbc:mysql://localhost:3306/study?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
   ##数据库用户名
   spring.datasource.username=root
   ##数据库密码
   spring.datasource.password=123456
   ```

7. 运行效果

### 5.2 JPA+Bootstrap

#### 5.2.1 Bootstrap

中文官网：https://www.bootcss.com/

下载Bootstrap之后将`bootstrap.min.css`放在resources.static.css目录下

将`bootstrap.min.js`放在resources.static.js目录下

1. 表单

   直接将表单代码放在html文件body里面

   ```html
   <form>
     <div class="form-group">
       <label for="exampleInputEmail1">Email address</label>
       <input type="email" class="form-control" id="exampleInputEmail1" placeholder="Email">
     </div>
     <div class="form-group">
       <label for="exampleInputPassword1">Password</label>
       <input type="password" class="form-control" id="exampleInputPassword1" placeholder="Password">
     </div>
     <div class="form-group">
       <label for="exampleInputFile">File input</label>
       <input type="file" id="exampleInputFile">
       <p class="help-block">Example block-level help text here.</p>
     </div>
     <div class="checkbox">
       <label>
         <input type="checkbox"> Check me out
       </label>
     </div>
     <button type="submit" class="btn btn-default">Submit</button>
   </form>
   ```

#### 5.2.2 复杂项目

1. 数据库

   ```sql
   CREATE TABLE `t_account` (
   	`id` INT(11) NOT NULL AUTO_INCREMENT,
   	`login_name` VARCHAR(50) NOT NULL DEFAULT '' COLLATE 'utf8_general_ci',
   	`password` VARCHAR(50) NOT NULL DEFAULT '' COLLATE 'utf8_general_ci',
   	`nick_name` VARCHAR(50) NOT NULL DEFAULT '' COLLATE 'utf8_general_ci',
   	`age` INT(11) NOT NULL DEFAULT '0',
   	`location` VARCHAR(50) NULL DEFAULT '',
   	PRIMARY KEY (`id`)
   )
   COLLATE='utf8mb4_general_ci'
   ENGINE=InnoDB
   ;
   ```
   
2. controller

   ```java
   @Controller
   public class MainController {
   
       @Autowired
       AccountService accSrv;
   
       @RequestMapping("/list")
       @ResponseBody
       public Object list() {
           // List<Account> list = accSrv.findAll();
           Object account = accSrv.findxxx();
           System.out.println("account:" + account);
           return account;
       }
   
       /**
        * 查数据
        * 区分 get 和 post 请求
        * <p>
        * get : 展示页面
        * post：收集数据
        *
        * @return
        */
       @GetMapping("/register")
       public String register(Model map) {
           System.out.println("======get=====");
           map.addAttribute("obj", "aa");
           return "register";
       }
   
   	/**
   	 * post请求，保存数据
   	 * @param request
   	 * @param account
   	 * @return
   	 */
       @PostMapping("/register")
       public String registerP(HttpServletRequest request, Account account) {
           System.out.println(account);
           RespStat stat = accSrv.save(account);
           request.setAttribute("stat", stat);
           return "register";
       }
   
       /**
        * 没有实际含义，刷新页面使用
        * @return
        */
       @RequestMapping("/login")
       public String login() {
           return "login";
       }
   }
   
   ```

3. service

   ```java
   @Service
   public class AccountService {
   
       @Autowired
       AccountRepository accRep;
   
       public RespStat save(Account account) {
           /**
            * 返回的实体类，id带回来。
            */
           try {
               Account entity = accRep.save(account);
           } catch (Exception e) {
               return new RespStat(500, "发生错误", e.getMessage());
           }
           return RespStat.build(200);
       }
   
       public List<Account> findAll() {
           // 查询所有数据
           //	return accRep.findAll();
           // 自定义方法
           return accRep.findByIdBetween(1, 6);
       }
   
       public Optional<Account> findById(int id) {
           // 接口自带方法
           return accRep.findById(id);
       }
   
       public Object findxxx() {
           List<Account> findbyxx = accRep.findbyxx2(1);
           return findbyxx;
       }
   }
   ```

4. dao

   ```java
   public interface AccountRepository extends JpaRepository<Account, Integer> {
   
       List<Account> findByIdBetween(int max, int min);
   
       Account findByLoginNameAndPassword(String loginName, String password);
   
       // 自定义 hql
       @Query("select acc from Account acc where acc.id=1 ")
       List<Account> findbyxx();
   
       @Query("select acc from Account acc where acc.id=?1 ")
       List<Account> findbyxx2(int id);
   }
   ```

5. entity

   ```java
   @Entity
   @Table(name = "t_account")
   public class Account {
   
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private int id;
   
       private String loginName;
       private String password;
       private String nickName;
       private int age;
       private String location;
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public String getLoginName() {
           return loginName;
       }
   
       public void setLoginName(String loginName) {
           this.loginName = loginName;
       }
   
       public String getPassword() {
           return password;
       }
   
       public void setPassword(String password) {
           this.password = password;
       }
   
       public String getNickName() {
           return nickName;
       }
   
       public void setNickName(String nickName) {
           this.nickName = nickName;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public String getLocation() {
           return location;
       }
   
       public void setLocation(String location) {
           this.location = location;
       }
   
       @Override
       public String toString() {
           return "Account{" +
                   "id=" + id +
                   ", loginName='" + loginName + '\'' +
                   ", password='" + password + '\'' +
                   ", nickName='" + nickName + '\'' +
                   ", age=" + age +
                   ", location='" + location + '\'' +
                   '}';
       }
   }
   ```

6. 标准返回类

   ```java
   public class RespStat {
   
       /**
        * JSON报文
        * 状态码
        * 用于前端判断   200  = 成功
        * 400\500 出错
        * <p>
        * msg = 信息
        */
   
       private int code;
       private String msg;
       private String data;
   
   
       public RespStat() {
           super();
       }
   
       public RespStat(int code, String msg, String data) {
           super();
           this.code = code;
           this.msg = msg;
           this.data = data;
       }
   
   
       public int getCode() {
           return code;
       }
   
       public void setCode(int code) {
           this.code = code;
       }
   
       public String getMsg() {
           return msg;
       }
   
       public void setMsg(String msg) {
           this.msg = msg;
       }
   
       public String getData() {
           return data;
       }
   
       public void setData(String data) {
           this.data = data;
       }
   
       public static RespStat build(int i) {
           return new RespStat(200, "ok", "meiyou");
       }
   }
   ```

7. register.html

   ```html
   <!DOCTYPE html>
   <html xmlns:th="http://www.w3.org/1999/xhtml">
   <head>
       <meta charset="UTF-8">
       <title>用户注册</title>
   
       <!-- 最新的 Bootstrap 核心 css 文件 -->
       <!-- 在url上 使用 @标签 可以帮我们 自动加上 contextpath -->
       <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
       <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
       <script th:src="@{/js/bootstrap.min.js}"></script>
       <script type="text/javascript" th:inline="javascript">
       </script>
   </head>
   <body>
   
   <p th:text="${stat == null ? '':stat.msg}" class="bg-danger">...</p>
   <p>
   
   </p>
   <hr>
   
   <a th:href="@{/list}">用户列表</a>
   <a th:href="@{/register}">用户注册</a>
   
   <form action="register" method="post">
       <div class="form-group">
           <label for="loginName">loginName</label>
           <input type="text" name="loginName" class="form-control" id="loginName" placeholder="请输入登录名">
       </div>
       <div class="form-group">
           <label for="Password">Password</label>
           <input type="password" name="password" class="form-control" id="Password" placeholder="请输入密码">
       </div>
       <div class="form-group">
           <label for="nickName">昵称</label>
           <input type="text" name="nickName" class="form-control" id="nickName" placeholder="你的昵称是？">
       </div>
       <div class="form-group">
           <label for="age">年龄</label>
           <input type="text" name="age" class="form-control" id="age" placeholder="你的年龄是？">
       </div>
       <button type="submit" class="btn btn-default">Submit</button>
   </form>
   
   </body>
   </html>
   ```

8. application.properties

   ```properties
   server.port=9000
   spring.datasource.url=jdbc:mysql://localhost:3306/study?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
   ##数据库用户名
   spring.datasource.username=root
   ##数据库密码
   spring.datasource.password=123456
   
   server.servlet.context-path=/account
   
   #显示sql
   spring.jpa.show-sql=true
   ```

## 6 Mybatis

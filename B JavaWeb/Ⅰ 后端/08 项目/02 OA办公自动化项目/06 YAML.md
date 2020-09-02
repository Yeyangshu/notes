# YAML

##  1 什么是YAML？

YAML是”YAML Ain't Markup Language YAML不是一种标记语言“的外语缩写

但为了强调这种语言以数据做为中心，而不是以置标语言为重点，而用返璞词重新命名。它是一种直观的能够被电脑识别的数据序列化格式，是一个可读性高并且容易被人类阅读，容易和脚本语言交互，用来表达资料序列的编程语言。

它是类似于标准通用标记语言的子集XML的数据描述语言，语法比XML简单很多。

## 2 YAML准则

1、大小写敏感 

2、使用缩进表示层级关系 

3、禁止使用 tab 缩进，只能使用空格键 

4、缩进长度没有限制，只要元素对齐就表示这些元素属于一个层级。 

5、使用#表示注释 

6、字符串可以不用引号标注

## 3 自定义参数

自定义参数可以让我们在配置文件中定义一些参数以供在程序中使用

在这里我们使用Spring注解的方式实现这个功能

首先创建一个**实体类**来保存一些我们的系统设置

```java
/**
 * 系统配置相关
 * @author Administrator
 *
 */
@Component
public class Config {

	@Value(value = "${config.systemName}")
	private String systemName;

	public String getSystemName() {
		return systemName;
	}

	public void setSystemName(String systemName) {
		this.systemName = systemName;
	}
}
```

application.properties中自定义系统变量

如果需要中文字符需要自行转换成Unicode

```
config.systemName=\u0078\u0078\u006f\u006f\u77f3\u5316\u662f\u6211\u5bb6
```

使用注解自动注入

```java
@Autowired
Config config;
```

### 3.1 参数引用

在application.propertie中的各个参数值是可以相互引用的

我们修改一下之前的配置

```properties
dalao.name=mashibing

dalao.yanzhi=100

dalao.desc=${dalao.name}is a good teacher,bing bu shi yin wei ${dalao.name} de yan zhi = ${dalao.yanzhi} 
```

### 3.2 随机数

有些特殊需求，我们不希望设置的属性值是一个固定值，比如服务器随机端口号，某些编号等，我们可以使用${radom}在配置中产生随机int，long或是string



${random.int()} = 随机int

${random.long} = 随机long

${random.int(50)} = 50以内的随机数

${random.int(50,100)} = 50~100之间的int随机数

${random.value}= 随机字符串

配置文件中使用

```properties
dalao.xiaodi.zhangyang.yanzhi=${random.int(50,100)}
dalao.xiaodi.zhangyang.xinqing=${random.value}
```

## 4 外部参入

## 5 优先级

**如果在properties里配置了属性，会覆盖yaml里的属性设置。**

在微服务架构中经常会使用自动运维部署工具，使用这些工具来启动我们的服务

我们的Spring boot程序通常是使用java –jar的方式来启动运行的

对于服务端口号或是一些其他需要在启动服务的时候才能决定的值，如果在配置中写死或是用随机明显是满足不了需求的

我们可以使用外部参数替换自定义的参数 

```
比如临时决定服务端口：
java -jar demo-0.0.1-SNAPSHOT.jar --server.port=60

颜值同时发生变化：
java -jar demo-0.0.1-SNAPSHOT.jar --server.port=60 --dalao.xiaodi.zhangyang.yanzhi=100
```

使用外部配置方式可以让我们在服务启动时改变像服务端口，数据库连接密码，自定义属性值等等 



## 6 多环境配置

在实际开发中，我们的一套代码可能会被同时部署到开发、测试、生产等多个服务器中，每个环境中诸如数据库密码等这些个性化配置是避免不了的，虽然我们可以通过自动化运维部署的方式使用外部参数在服务启动时临时替换属性值，但这也意味着运维成本增高。

我们可以通过多套配置来避免对于不同环境修改不同的配置属性

命名规则为：

```properties
application-*.properties
application-dev.properties = 开发环境
application-test.properties = 测试环境
application-prod.properties = 生产环境
```

接下来我们在 application.properties中设置哪套配置生效的开关

使用 **spring.profiles.active=dev**

在使用 java –jar 的方式启动服务的时候我们就可以通过外部参数改变整套配置了

**java -jar demo-0.0.1-SNAPSHOT.jar -- spring.profiles.active=test**

## 7 使用YAML完成多环境配置

### 方式一：单一yml文件 配合多propertys文件

```yaml
spring:
  profiles:
    active:
      - prod
```

### 方式二：单一yml文件内配置所有变量

```yaml
server:     
  port: 8881 
spring:
  profiles:
     active:
      - prod

 --- 

spring:     
  profiles: test 
server:     
  port: 8882 

dalao:
  name: mashibing
  yanzhi: 100
  desc: ${dalao.name}is a good teacher,\啊啊啊  bing bu shi yin wei ${dalao.name} de yan zhi = ${dalao.yanzhi} 

  

 ---

spring:
  profiles: dev 
server:     
  port: 8082
dalao:
  name: mashibing
  yanzhi: 100
  desc: ${dalao.name}is a good teacher,开发 bing bu shi yin wei ${dalao.name} de yan zhi = ${dalao.yanzhi} 

 ---

spring:
  profiles: prod 
server:     
  port: 8083
dalao:
  name: mashibing
  yanzhi: 100
  desc: ${dalao.name}is a good teacher,生产  bing bu shi yin wei ${dalao.name} de yan zhi = ${dalao.yanzhi}
```



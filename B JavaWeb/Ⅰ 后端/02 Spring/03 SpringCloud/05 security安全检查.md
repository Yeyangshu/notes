## 安全配置

### 开启Eureka安全连接

```
spring.security.user.name=yiming
spring.security.user.password=123
```



![image-20200408185532993](C:\Study\InternetArchitect-master\20 架构师三期 SpringCloud微服务架构\images\image-20200408185532993.png)



### 如果服务注册报错

```
Root name 'timestamp' does not match expected ('instance') for type [simple
```

是默认开启了防止跨域攻击



#### 手动关闭

在服务端增加配置类

```
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

## 服务间调用

		微服务中，很多服务系统都在独立的进程中运行，通过各个服务系统之间的协作来实现一个大项目的所有业务功能。服务系统间 使用多种跨进程的方式进行通信协作，而RESTful风格的网络请求是最为常见的交互方式之一。

http。

	思考：如果让我们写服务调用如何写。

1. 硬编码。不好。ip域名写在代码中。目的：找到服务。

2. 根据服务名，找相应的ip。目的：这样ip切换或者随便变化，对调用方没有影响。

   Map<服务名，服务列表> map;

3. 加上负载均衡。目的：高可用。



spring cloud提供的方式：

1. RestTemplate
2. Feign

我个人习惯用RestTemplate，因为自由，方便调用别的第三方的http服务。feign也可以，更面向对象一些，更优雅一些，就是需要配置。

### Rest

## 

```sh
RESTful网络请求是指RESTful风格的网络请求，其中REST是Resource Representational State Transfer的缩写，直接翻译即“资源表现层状态转移”。
Resource代表互联网资源。所谓“资源”是网络上的一个实体，或者说网上的一个具体信息。它可以是一段文本、一首歌曲、一种服务，可以使用一个URI指向它，每种“资源”对应一个URI。
Representational是“表现层”意思。“资源”是一种消息实体，它可以有多种外在的表现形式，我们把“资源”具体呈现出来的形式叫作它的“表现层”。比如说文本可以用TXT格式进行表现，也可以使用XML格式、JSON格式和二进制格式；视频可以用MP4格式表现，也可以用AVI格式表现。URI只代表资源的实体，不代表它的形式。它的具体表现形式，应该由HTTP请求的头信息Accept和Content-Type字段指定，这两个字段是对“表现层”的描述。
State Transfer是指“状态转移”。客户端访问服务的过程中必然涉及数据和状态的转化。如果客户端想要操作服务端资源，必须通过某种手段，让服务器端资源发生“状态转移”。而这种转化是建立在表现层之上的，所以被称为“表现层状态转移”。客户端通过使用HTTP协议中的四个动词来实现上述操作，它们分别是：获取资源的GET、新建或更新资源的POST、更新资源的PUT和删除资源的DELETE。
```

RestTemplate是Spring提供的同步HTTP网络客户端接口，它可以简化客户端与HTTP服务器之间的交互，并且它强制使用RESTful风格。它会处理HTTP连接和关闭，只需要使用者提供服务器的地址(URL)和模板参数。

```
第一个层次（Level 0）的 Web 服务只是使用 HTTP 作为传输方式，实际上只是远程方法调用（RPC）的一种具体形式。SOAP 和 XML-RPC 都属于此类。
第二个层次（Level 1）的 Web 服务引入了资源的概念。每个资源有对应的标识符和表达。
第三个层次（Level 2）的 Web 服务使用不同的 HTTP 方法来进行不同的操作，并且使用 HTTP 状态码来表示不同的结果。如 HTTP GET 方法来获取资源，HTTP DELETE 方法来删除资源。
第四个层次（Level 3）的 Web 服务使用 HATEOAS。在资源的表达中包含了链接信息。客户端可以根据链接来发现可以执行的动作。
```

**git的restful api**

https://developer.github.com/v3/


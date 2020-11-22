# Servlet

## 1 Servlet简介

- 是一种Web服务器端编程技术。

- 是实现了特殊接口的Java类。

- 由支持Servlet的Web服务器调用和启动运行。

- 一个Servlet负责对应的一个或一组URL访问请求，并返回相应的响应内容。

### 1.1 实现使用

- 创建一个普通java文件

- Java文件的类名实现HttpServlet

- 重写service的方法

- 在WEB-INF下的web.xml中添加请求与servlet类的映射关系

### 1.2 Servlet特点

- 运行在支持java的应用服务器上

- 服务器能根据请求调用对应的servlet进行请求处理

- 简单方便，可移植性强

### 1.3 Servlet运行流程

![image-20201122234158802](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122234158802.png)

### 1.4 Servlet的访问流程

url：http://localhost:8080/firstweb/first

组成：

- 服务器地址：端口/虚拟项目名/servlet的别名

- uri：虚拟项目名/servlet别名

- 过程：浏览器发送请求到服务器，服务器根据请求URL地址中的URI信息在webapps目录下找到对应的项目文件夹，然后再web.xml中检索对应的servlet，找到后调用并执行servlet

### 1.5 Servlet的生命周期

![image-20201122234306117](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122234306117.png)

### 1.6 Service、doGet、doPost方法的区别

Service方法

- 不管是get还是post请求方式，如果service方法存在，则优先执行service方法

doGet方法

- 在没有service的情况下，如果是get请求，调用doGet方法

doPost方法

- 在没有service的情况下，如果是post请求，调用diPost方法

### 1.7 Servlet常见错误

404：访问资源不存在

- 请求路径跟web.xml中填写的请求不一致

- 请求路径的项目虚拟名称填写错误

405：

- 请求的方式跟servlet中支持的方式不一致

500：服务器内部错误

- web.xml中servlet类的名称错误

- servlet对应的处理方法中存在代码逻辑错误

## 2 Request和Response

### 2.1 HttpServletRequest

HttpServletRequest对象代表客户端的请求，当客户端通过HTTP协议访问服务器时，HTTP请求中的所有信息都封装在这个对象中，通过此种方式来获取请求中的数据

#### 2.1.1 request常用方法

- getRequestURL：获取客户端的完整URL

- getRequesURI：获取请求行中的资源名部分

- getQueryString：获取请求行的参数部分

- getMethod：获取请求方式

- getSchema：获取请求的协议

- getRemoteAddr：获取客户端的ip地址

- getRemoteHost：获取客户端的完整主机名

- getRemotePort：获取客户端的网络端口号

- 获取请求头信息
  - getHeader(String name)
  - getHeaders(String name)

- 获取客户端请求参数（用户提交的数据）
  - getParameter(name)
  - getParameterValues(String name)
  - getParameterNames()
  - getParameterMap()

### 2.2 HttpServletResponse

HttpServletResponse对象是服务器的响应对象，这个对象中封装了向客户端发送数据，发送响应头，发送响应状态码的方法。

#### 2.2.1 response常用方法

- 设置响应头
  - setHeader(String key,String value) 添加响应信息，key重复会覆盖
  - addHeader(String key,String value) 添加响应信息，key重复不会覆盖

- 设置响应状态
  - sendError(int num,String msg ) 添加响应状态

- 设置响应实体
  - getWriter().write(msg)  响应具体的数据给浏览器

### 2.3 乱码问题解决

1. 使用String重新进行编码
   - String name = new String(name.getBytes(“ios-8859-1”),”utf-8”)

2. get请求乱码
   - request.setCharacterEncoding("utf-8");
   - 在server.xml中添加属性useBodyEncodingForURI=true

3. post请求乱码
   - request.setCharacterEncoding("utf-8");

4. response乱码
   - response.setCharacterEncoding("UTF-8");
   - response.setContentType("text/html;charset=utf-8")

### 2.4 Servlet的流程处理

![image-20201122235633596](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122235633596.png)



### 2.5 Servlet请求转发

![image-20201122235733468](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122235733468.png)

①客户端只发送一次请求

②浏览器的地址栏地址没有变化

③请求过程中只有一个request和response

④几个servlet共享一个request和response

⑤对客户端透明，客户端不需要知道服务端做了哪些操作

### 2.6 Servlet request作用域

思考：不同的servlet之间如何实现数据共享？

用法：

- request.setAttribute(Object key,Object value)

- request.getAttribute(Object key)

### 2.7 Servlet重定向

![image-20201122235949559](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122235949559.png)

①浏览器发送两次请求

②浏览器的地址发生变化

③请求过程产生两个request对象和两个response对象

④两个servlet不能共享同一个request和response对象

### 2.8 请求转发和重定向的区别

![image-20201123000028538](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201123000028538.png)


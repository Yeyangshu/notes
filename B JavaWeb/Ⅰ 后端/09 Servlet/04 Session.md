# Session

思考：一个用户的不同请求的处理需要使用相同的数据怎么办？

使用session来处理

## 1 什么是Session？

### 1.1 概念

- Session表示会话，在一段时间内，用户与服务器之间的一系列的交互操作。

- session对象：用户发送不同请求的时候，在服务器端保存不同请求共享数据的存储对象

### 1.2 特点

- Session是依赖cookie技术的服务器端的数据存储技术

- 由服务器进行创建

- 每个用户独立拥有一个session对象

- 默认存储时间是30分钟

## 2 Session机制

用户使用浏览器第一次向服务器发送请求， 服务器在接受到请求后， 调用对应的 Servlet 进行处理。 在处理过程中会给用户创建一个 session 对象， 用来存储用户请求处理相关的公共数据， 并将此 session 对象的 JSESSIONID 以 sessoin 的形式存储在浏览器中(临时存储， 浏览器关闭即失效)。 用户在发起第二次请求及后续请求时， 请求信息中会附带 JSESSIONID， 服务器在接收到请求后，调用对应的 Servlet 进行请求处理， 同时根据 JSESSIONID 返回其对应的 Session对象

## 3 Session的基本原理

![image-20201123000753862](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201123000753862.png)

### 3.1 Session基本操作

- 创建Session对象
  - HttpSession session = request.getSession()

- 向Session对象中添加值
  - sessoin.setAttribute(String name,Object object)

- 获取Session中的值
  - session.getAttribute(String name)

- 设置Session属性
  - session.setMaxInactiveInterval(5)//设置存活时间
  - session.invalidate(); //session强制失效
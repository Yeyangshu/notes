# Cookie

## 1 无状态的HTTP协议

- HTTP是一个无状态的协议，当一个客户端向服务端发送请求，在服务器返回响应后，连接就关闭了，在服务器端不保留连接信息。

思考：当客户端发送多次请求且需要相同的请求参数的时候，应该如何处理？

## 2 什么是Cookie?

- Cookie是一种在客户端保持HTTP状态信息的技术

- Cookie是在浏览器访问服务器的某个资源时，由web服务器在响应头传送给浏览器的数据

- 浏览器如果保存了某个cookie，那么以后每次访问服务器的时候，都会在请求头传递给服务端

- 一个cookie只能记录一种信息，是key-value形式

- 一个web站点可以给浏览器发送多个cookie，一个浏览器也可以存储多个站点的cookie

## 3 Cookie的基本原理

![image-20201123000434413](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201123000434413.png)

- 创建cookie对象
  - Cookie cookie = new Cookie(String key,String value);

- 设置cookie参数
  - cookie.setMaxAge(int seconds)
  - cookie.setPath(String url)

- 响应Cookie给客户端
  - response.addCookie(Cookie cookie)

- 获取cookie属性
  - Cookie[] cookies = request.getCookies()
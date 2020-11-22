# HTTP协议

## 1 客户端与服务端的交互原理

![image-20201122232838388](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122232838388.png)

客户端的浏览器版本很多，如何实现不同版本的浏览器与服务器的通信？

## 2 HTTP协议

超文本传输协议(Hyper Text Transfer Protocol)

作用：规范了浏览器和服务器的数据交互

特点：

- 简单快速
- 灵活
- 无连接
- 无状态
- 支持B/S和C/S架构

注意：HTTP1.1版本之后支持可持续连接

### 2.1 HTTP协议的交互流程

![image-20201122233055103](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122233055103.png)

### 2.2 HTTP协议请求格式

![image-20201122233135984](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122233135984.png)

### 2.3 HTTP请求方法

| **方法** |                       **作用**                       |
| :------: | :--------------------------------------------------: |
|   GET    |          请求获取由Request-URI所标识的资源           |
|   POST   |       在Request-URI所标识的资源后附件新的数据        |
|   HEAD   |   请求获取由Request-URI所标识的资源的响应消息报头    |
|  DELETE  |        请求服务器删除由Reqest-URI所标识的资源        |
|  TRACE   |     请求服务器会送收到的请求信息，用于测试或诊断     |
| CONNECT  |                     保留将来使用                     |
| OPTIONS  | 请求查询服务器的性能，或者查询与资源相关的选项和需求 |
|   PUT    |  请求服务器存储一个资源，并用Request-URI作为其标识x  |

GET和POST请求方式的区别：

1. get请求参数是直接显示在地址栏的，而post在地址栏不显示

2. get方式不安全，post安全

3. get请求参数是又长度限制的，post没有限制

### 2.4 HTTP协议响应格式

![image-20201122233508826](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122233508826.png)

| **分类** |                  **分类描述**                  |
| :------: | :--------------------------------------------: |
|   1**    |  信息，服务器收到请求，需要请求者继续执行操作  |
|   2**    |           成功，操作被成功接受并处理           |
|   3**    |       重定向，需要进一步的操作以完成请求       |
|   4**    |   客户端错误，请求包含语法错误或无法完成请求   |
|   5**    | 服务器操作，服务器在处理请求的过程中发生了错误 |

常见的响应状态码：

- 200 OK //客户端请求成功

- 400 Bad Request //客户端请求有语法错误，不能被服务器所理解

- 401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate 报头域一起使用

- 403 Forbidden //服务器收到请求，但是拒绝提供服务

- 404 Not Found //请求资源不存在，eg：输入了错误的 URL

- 500 Internal Server Error //服务器发生不可预期的错误

- 503 Server Unavailable //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
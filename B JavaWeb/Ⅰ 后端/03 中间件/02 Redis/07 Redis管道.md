# 管道

管道（Pipelining）：学习如何一次发送多个命令，节省往返时间。

先看官网链接：http://redis.cn/topics/pipelining.html



# 1 nc

### 1.1 nc的作用

1. 实现任意TCP/UDP端口的侦听，nc可以作为server以TCP或UDP方式侦听指定端口
2. 端口的扫描，nc可以作为client发起TCP或UDP连接
3. 机器之间传输文件
4. 机器之间网络测速 

### 1.2 nc安装

```
yum install nc
nc localhost 6379
```

## 2 管道

一次请求/响应服务器能实现处理新的请求即使旧的请求还未被响应。这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。

### 2.1 管道的使用

 ```
[root@node1 init.d]# echo -e "set k2 99\nincr k2\nget k2" | nc localhost 6379
+OK
:100
$3
100
 ```


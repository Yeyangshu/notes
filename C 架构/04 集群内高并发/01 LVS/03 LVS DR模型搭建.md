# LVS DR模型搭建

## 1 基础知识

### 1.1 隐藏VIP方法：对外隐藏，对内可见

kernel parameter

目标地址全为F，交换机触发广播

/proc/sys/net/ipv4/*IF*/

```txt
arp_ignore：定义接收到APP请求时的响应级别：

0：只要本地配置有相应地址，就给予响应
1：仅在请求的目标（MAC）地址配置请求到达的接口上的时候，才给予响应

arp_announce：定义将自己地址向外通告时的通告级别：
0：将本地任何接口上的任何地址向外通告
1：识图仅向目标网络通告与其网络匹配的地址
2：仅向与本地接口上地址匹配的网络进行通告
```

![image-20200712110715535](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712110715.png)

![image-20200712110831977](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712110832.png)

### 1.2 调度算法

调度算法分为静态调度和动态调度

#### 1.2.1 静态调度

- rr：轮询
- wrr：
- dh：
- sh：

#### 1.2.2 动态调度

- lc：最少连接
- wlc：加权最少连接
- sed：最短期望延迟
- nq：never queue
- LBLC：基于本地的最少连接
- DH：
- LBLCR：基于本地的带复制功能的最少连接

### 1.3 Linux内核模块

LVS在Linux系统叫ipvs，Linux系统默认自带ipvs

普通用户需要用户空间程序，安装交互接口软件

```
yum install ipvsadm -y
```

#### 1.3.1 管理集群服务

- 添加：-A -t|u|f service-address [-s sccheuler]

  ```
  -t：TCP协议的集群
  -u：UDP协议的集群
  service-address：ip:port
  -f：防火墙标记
  service-address：MARK NUMBER
  ```

- 修改：-E

- 删除：-D -t|u|f service-address [-s sccheuler]

样例

```
ipvsadm -A -t 192.168.9.100:80 -s rr
```

#### 1.3.2 管理集群中的RS

- 添加：-a -t|u|f service-address -r sever-address [-g|i|m] [-w weight]

  ```
  -t|u|f service-address：实现定义好的某集群服务；
  -r sever-address：某RS的地址，在NAT模型中，可使用ip:port实现端口映射；
  [-g|i|m]：LVS类型
    -g：DR
    -i：TUN
    -m：NAT
  [-w weight]：定义服务器权重
  ```

- 修改：-e

- 删除：-d -t|u|f service-address -r sever-address

  ```
  ipvsadm -d -t 192.168.163.100:80 -r 192.168.163.111
  ```

  

添加样例

```
ipvsadm -a -t 172.16.100.1:80 -r 192.168.10.8 -g
ipvsadm -a -t 172.16.100.1:80 -r 192.168.10.9 -g
```

- 查看

  -L|I

  -n：数字显示主机地址和端口

  --stats：统计数据

  --rate：速率

  --timeout：显示tcpfin和udp的会话超时时长

  --c：显示当前的ipvs连接情况

- 删除所有集群服务
  -C：清空ipvs规则

- 保存规则
  -S

  ```
  ipvsadm -S > /path/to/somefile
  ```

- 载入此前的规则
  -R

  ```
  ipvsadm -R < /path/to/somefile
  ```

## 2 实验手册

![image-20200712153443172](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712153443.png)

### 2.1 网络层

#### 2.1.1 

1. 更改node1

   ```
   ifconfig eth0:2 192.168.163.100/24
   等于
   ifconfig eth0:2 192.168.163.100 netmask 255.255.255.0
   
   如果需要废弃
   ifconfig eth0:2 down
   ```

   ![image-20200712155038366](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712155038.png)

2. node2/3 修改内核
   ![image-20200712160609523](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712160609.png)

   返回all目录继续修改

   ![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712160755.png)

   ```
   echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
   echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
   echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce 
   echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
   ```

   

3. node2/3 设置隐藏的vip

   ```
   ifconfig lo:8 192.168.163.100 netmask 255.255.255.255
   ```

   

   ![image-20200712161219390](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712161219.png)

   

#### 2.1.2 RS中的服务

1. node2/3 安装httpd

   ```
   yum install httpd -y
   安装完
   service httpd start
   ```

2. 创建主页

   ```
   vi /var/www/html/index.html
   ```

   访问浏览器

   ![image-20200712164208061](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712164208.png)

#### 2.1.3 LVS服务配置

1. 添加ipvsadm

   ```
   yum install ipvsadm -y
   ```

2. 添加入口规则

   ```
   # ipvsadm -A -t 192.168.163.100:80 -s rr
   # ipvsadm -ln
   IP Virtual Server version 1.2.1 (size=4096)
   Prot LocalAddress:Port Scheduler Flags
     -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   TCP  192.168.163.100:80 rr
   ```

3. 出口规则

   ```
   # ipvsadm -a -t 192.168.163.100:80 -r 192.168.163.112 -g -w 1
   # ipvsadm -a -t 192.168.163.100:80 -r 192.168.163.113 -g -w 1
   
   # ipvsadm -ln
   IP Virtual Server version 1.2.1 (size=4096)
   Prot LocalAddress:Port Scheduler Flags
     -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   TCP  192.168.163.100:80 rr
     -> 192.168.163.112:80           Route   1      0          0         
     -> 192.168.163.113:80           Route   1      0          0      
   ```

#### 2.1.5 验证

1. node1验证

   ```
   netstat -natp
   ```

   结论，看不到socket连接

   ![image-20200712193254827](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712193255.png)

2. node2验证

   有很多socket连接

   ![image-20200712193425968](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712193426.png)

3. node1验证

   查看偷窥记录本

   ```
   ipvsadm -lnc
   ```

   ![image-20200712193614906](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/20200712193615.png)

   FIN_WAIT：连接过，偷窥了所有的包

   SYN_RECV：基本上lvs都记录了，证明lvs没事，一定是后边网络层出问题
   		down掉112，继续访问100，可以验证此种情况


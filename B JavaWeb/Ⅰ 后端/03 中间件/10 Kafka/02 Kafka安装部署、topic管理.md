# Kafka部署

Kafka官方网址点击 -> DOWNLOAD KAFKA

![image-20201122104420525](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201122104420525.png)

## 1 环境搭建-单机

1. 安装JDK1.8+，配置JAVA_HOME (**CentOS 6.10 64bit)**  

2. 配置主机名和IP映射

3. 关闭防火墙&防火墙开机自启动

   ```shell
   # 查询防火墙状态
   [root@node1 soft]# service iptables status
   iptables: Firewall is not running.
   # 关闭防火墙
   service iptables stop
   ```

4. 安装&启动Zookeeper

   ```properties
   zkServer.sh start
   ```

5. 安装&启动|关闭Kafka

   ```properties
   # 解压文件到指定的目录
   tar -zxvf kafka_2.11-2.2.0.tgz -C /opt/soft
   # 进入指定文件夹
   cd /opt/soft/kafka_2.11-2.2.0
   # 此时所在目录
   pwd
   /opt/soft/kafka_2.11-2.2.0
   # 修改配置文件
   vi config/server.properties
   #############################
   # 单机不需要修改
   broker.id=0
   # 改成主机名
   listeners=PLAINTEXT://node01:9092
   # 日志存储文件夹
   log.dirs=/usr/kafka-logs
   # zookeeper服务
   zookeeper.connect=node01:2181
   #############################
   # 启动kafka服务
   ./bin/kafka-server-start.sh -daemon config/server.properties 
   # 关闭kafka
   ./bin/kafka-server-stop.sh
   # 判断kafka是否程启动
   jps
   4149 Kafka
   4170 Jps
   3789 QuorumPeerMain
   
   /opt/soft
   ```

## 2 集群



## 3  Topic管理



```properties
# topic帮助
./bin/kafka-topics.sh --help

[root@node1 kafka_2.11-2.2.0]# ./bin/kafka-topics.sh --bootstrap-server node01:9092 --create --topic topic01 --partitions 3 --replication-factor 1
```












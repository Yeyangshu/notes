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
   ```

单机测试图：

![image-20201123224745591](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201123224745591.png)

## 2 集群

- 安装JDK，配置JAVA_HOME (**CentOS 6.10 64bit)**  

- 配置主机名和IP映射

  ```
  192.168.163.111 node01
  192.168.163.112 node02
  192.168.163.113 node03
  192.168.163.114 node04
  ```

- 关闭防火墙&防火墙开机自启动

- 同步时钟 ntpdate cn.pool.ntp.org **| ntp[1-7].aliyun.com**

  ```shell
  yum install ntp -y
  ntpdate ntp1.aliyun.com
  clock -w
  ```

- 安装&启动Zookeeper

- 安装&启动|关闭Kafka

  ```properties
  # Kafka配置文件
  broker.id=4
  zookeeper.connect=node02:2181,node03:2181,node04:2181
  
  # 复制kafka文件到其他服务器
  scp -r kafka_2.11-2.2.0 node02:/opt/soft
  # 更改配置文件
  broker.id=3
  
  ```

## 3  Topic管理

```properties
# topic帮助
./bin/kafka-topics.sh --help
# 创建
[root@node1 kafka_2.11-2.2.0]# ./bin/kafka-topics.sh --bootstrap-server 192.168.163.111:9092 --create --topic topic01 --partitions 3 --replication-factor 1
# 订阅
[root@node1 kafka_2.11-2.2.0]# ./bin/kafka-console-consumer.sh --bootstrap-server 192.168.163.114:9092 --topic topic01 --group group01
# 生产
[root@node1 kafka_2.11-2.2.0]# ./bin/kafka-console-producer.sh --broker-list 192.168.163.114:9092 --topic topic01

```



- 创建

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh 
                      --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                      --create 
                      --topic topic02 
                      --partitions 3 
                      --replication-factor 3
  
  ```

- 查看

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 --list
  
  ./bin/kafka-topics.sh --bootstrap-server node02:9092,node03:9092,node04:9092 --list
  ```
  
- 详情

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh 
                      --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                      --describe 
                      --topic topic01
  Topic:topic01	PartitionCount:3	ReplicationFactor:3	Configs:segment.bytes=1073741824
  	Topic: topic01	Partition: 0	Leader: 0	Replicas: 0,2,3	Isr: 0,2,3
  	Topic: topic01	Partition: 1	Leader: 2	Replicas: 2,3,0	Isr: 2,3,0
  	Topic: topic01	Partition: 2	Leader: 0	Replicas: 3,0,2	Isr: 0,2,3
  
  ```

- 修改

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh 
                      --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                      --create 
                      --topic topic03 
                      --partitions 1 
                      --replication-factor 1
  
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh 
                      --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                      --alter 
                      --topic topic03 
                      --partitions 2
  ```

- 删除

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-topics.sh 
                      --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                      --delete 
                      --topic topic03
  
  ```

- 订阅

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-console-consumer.sh 
                    --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                    --topic topic01 
                    --group g1 
                    --property print.key=true 
                    --property print.value=true 
                    --property key.separator=,
  
  ```

- 生产

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-console-producer.sh 
                    --broker-list CentOSA:9092,CentOSB:9092,CentOSC:9092 
                    --topic topic01
  
  ```

- 消费组

  ```shell
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-consumer-groups.sh 
                    --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                    --list
                    g1
  
  [root@CentOSA kafka_2.11-2.2.0]# ./bin/kafka-consumer-groups.sh 
                    --bootstrap-server CentOSA:9092,CentOSB:9092,CentOSC:9092 
                    --describe 
                    --group g1
  
  TOPIC PARTITION CURRENT-OFFSET LOG-END-OFFSET LAG CONSUMER-ID    HOST            CLIENT-ID
  topic01 1                      0                    0                           0     consumer-1-**    /192.168.52.130 consumer-1
  topic01 0                      0                    0                          0      consumer-1-**   /192.168.52.130 consumer-1
  topic01 2                      1                     1                          0      consumer-1-**   /192.168.52.130 consumer-1
  ```

  

# 本章小结

- 掌握Kafka单机&集群搭建

- topic、partition、replication-factor、consumer group

- 消息publish/subscribe
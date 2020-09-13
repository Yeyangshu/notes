# 基于keepalived的LVS高可用搭建

## 1 学习keepalived前的问题

**问题：**

1. LVS负载均衡器故障，业务下线，单点故障
2. real server 故障，健康问题，一部分用户会请求异常，lvs还存有这个RS 的负载记录



**如何解决问题：**

单点故障的解决方式：lvs 是一个服务器，一个有问题，就可以使用一堆，一变多！

2个思路：多点，形式：

1. 主备
2. 主主



**结论：**

先讨论主备

方向性：主向备发包，一段时间内备没有收到主就可以认为主挂掉

效率型：权重，选权重最大的（皇子中的大哥）



两个概念

主备，主（单点->主备）从

RS挂了怎么确定？如何确定百度挂了？

错误回答：ping。

正确回答：访问一下 -> 底层原理：验证的是应用层的http协议 -> 发请求，判断返回，200 ok



lvs是内核中有模块：ipvs <- 增加代码判定？ 第三方实现？ <- 如何判定lvs挂了？？

代码有局限性，有可能不可访问，采用第三方

第三方：人做响应，最不靠谱

企业追求自动化，把人解耦出去，用程序替代！-> 这个程序就是keepalived

keepalived可以代替人自动运维，解决单点故障，实现HA

作用：

1. 监控自己服务
2. Master通告自己还活着，Backup监听Master状态，Master挂了，一堆Backup推举出一个新的Master
3. 配置vip。为vip地址所在的节点生成ipvs规则(在配置文件中预先定义)
4. 对后端RS 做健康检查

keepalived是一个通用的工具，主要作为HA实现

Nginx可以作为公司的负载均衡来用，Nginx成为了单点故障，也可以用keepalived来解决



## 2 keepalived实验

### 2.1 node01原来的配置

```
[root@node1 data]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.163.100:80 rr
  -> 192.168.163.112:80           Route   1      0          9         
  -> 192.168.163.113:80           Route   1      0          10      
```

ifconfig

```
[root@node1 data]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:DD:00:9D  
          inet addr:192.168.163.111  Bcast:192.168.163.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fedd:9d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1147558 errors:1467 dropped:1544 overruns:0 frame:0
          TX packets:640035 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1363371297 (1.2 GiB)  TX bytes:294003163 (280.3 MiB)
          Interrupt:19 Base address:0x2000 

eth0:2    Link encap:Ethernet  HWaddr 00:0C:29:DD:00:9D  
          inet addr:192.168.163.100  Bcast:192.168.163.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          Interrupt:19 Base address:0x2000 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:7662136 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7662136 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:814624025 (776.8 MiB)  TX bytes:814624025 (776.8 MiB)

```

### 2.2 node01删除配置

删除ipvsadm配置

```
[root@node1 data]# ipvsadm -C
[root@node1 data]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
```

删除vip配置

```
ifconfig eth0:2 down
```

### 2.3 node01、node04安装keepalived

安装命令

```
yum install keepalived ipvsadm -y
```

配置文件位置

```
/etc/keepalived/keepalived.conf
```

备份一下文件

```
cp keepalived.conf keepalived.conf.bak
```



vrrp：虚拟路由冗余协议

node01修改配置

```
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
	192.168.163.100/24 dev eth0 label eth0:3  // 配置vip
    }
}

virtual_server 192.168.163.100 80 { // 配置real server
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0
    persistence_timeout 0
    protocol TCP

    real_server 192.168.163.112 80 {  // 配置real server01
        weight 1
        HTTP_GET {
            url {
                path /
	      		status_code 200 // 健康检查
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.163.113 80 { // 配置real server02
        weight 1
        HTTP_GET {
            url {
              path /
	      status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

远程拷贝配置文件到node04

```
scp ./keepalived.conf root@node04:`pwd`
```

node04

```
vrrp_instance VI_1 {
    state BACKUP	// 改成BACKUP
    interface eth0
    virtual_router_id 51
    priority 50		//权重改成50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.163.100/24 dev eth0 label eth0:3
    }
}
```

附录知识点：

linux帮助命令

```
yum install man
man 5 keepalived.conf
```

![image-20200912212813812](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200912212813812.png)

### 2.4 启动keepalived

启动命令

```
service keepalived start
```

先启动node01，观察 ifconfig和ipvsadm -ln

keepalived已经配置好了vip和RS

![image-20200913094243999](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913094243999.png)

再启动node04 的服务观察 ifconfig和ipvsadm -ln

由于node04是备机，vip还没有，RS已经配置好

![image-20200913094419088](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913094419088.png)

### 2.5 验证HA

#### 2.5.1 主备切换

down掉node01 的网卡

```
ifconfig eth0 down
```

观察node04的ifconfig，已经有eth0:3，可以做负载

![image-20200913094905671](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913094905671.png)

再将 node01 的网卡启动

```
ifconfig eth0 up
```

node04 vip已经卸载

![image-20200913095253535](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913095253535.png)

node01 将vip抢回来

![image-20200913095540287](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913095540287.png)



注意事项：有些情况（不是keepalived）主不一定能抢的回来，备改成主，主改成备

#### 2.5.2 RS挂掉

node01显示node02和node03都能正常服务

![image-20200913100003359](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100003359.png)

关闭node02

```
service httpd stop
```

此时node01只显示一台

![image-20200913100053190](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100053190.png)

服务永远定位在node03

![image-20200913100154966](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100154966.png)

备机node04健康检查也会删除node02

![image-20200913100252225](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100252225.png)



此时恢复node02

```
service httpd start
```

node01和node04健康检查都会恢复node02

![image-20200913100417560](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100417560.png)

所有的操作对用户都是透明的

### 2.6 异常情况

keepalived只是一个程序，也会有异常退出进程的时候

直接杀死node01 的所有进程

```
kill -9 xxx
```

此时node01属于异常退出，node04就会启动配置，添加vip

![image-20200913100935905](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200913100935905.png)

两台机器都有vip的配置就会造成数据包的混乱


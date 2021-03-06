# Redis集群

## 1 前景

### 1.1 单点容量问题

前面的文章讲解的是Redis的主从复制和高可用，属于AKF的x轴，遗留一个y、z轴问题：单节点的容量的问题

### 1.2 解决方案

#### 1.2.1 客户端按业务逻辑拆分，不同业务访问不同的Redis

![image-20200722214752093](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722214752093.png)

#### 1.2.2 客户端按算法拆分，分片

1. modula

   ![image-20200722220408836](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722220408836.png)

2. random

   适用于消息队列，后面有一台Redis客户端接受消息

   ![image-20200722220521290](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722220521290.png)

3. kemata

   更适用于缓存使用，而不是数据库！

   ![image-20200722222506819](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722222506819.png)

   物理机node1和node2形成一个哈希环，数据会去下一个物理节点寻找

   ![image-20200722222752218](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722222752218.png)

   如果此时多添加一个node3物理机

   ![image-20200722222850964](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200722222850964.png)

   优点：

   - 加节点，的确可以分担其他节点的压力，不会造成全局洗牌

   缺点：

   - 新增节点造成一小部分数据不能命中

   问题：

   - 缓存击穿，数据访问压到mysql

   方案：

   - 每次去取离数据最近的2个物理节点寻找

#### 1.2.3 代理服务器集群

Redis连接成本很高

![image-20200724225051612](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200724225051612.png)

解决方式：负载均衡服务器，将客户端实现的算法转移至代理服务器

![image-20200724225619237](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200724225619237.png)



Nginx代理不作为数据库存储，所以是无状态的，可以做集群

![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200724225745033.png)



## 2 Redis分片

twitter教程：https://github.com/twitter/twemproxy

predixy：https://github.com/joyieldInc/predixy

代理

tw

predixy

cluster

codis

弊端：上面的三个模式不能做数据库使用

Redis集群

![image-20200725101257961](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200725101257961.png)

每个Redis服务器有其它服务器的映射关系（主主模型）

![image-20200725101339130](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200725101339130.png)

弊端：

数据分治

聚合操作很难实现

事务



http://redis.cn/topics/partitioning.html



## 3 Redis分片实战

### 3.1 twemproxy

soft目录新建推特代理文件夹

```
mkdir twemproxy
cd twemproxy
```

会报如下错误

>```
>[root@node1 twemproxy]# git clone https://github.com/twitter/twemproxy.git
>Initialized empty Git repository in /root/soft/twemproxy/twemproxy/.git/
>error:  while accessing https://github.com/twitter/twemproxy.git/info/refs
>
>fatal: HTTP request failed
>```
>
>解决方案：强制升级
>
>yum update nss

需要先安装`automake`和`libtool`

```
yum install automake libtool -y
```

按照官网继续

```
autoreconf -fvi
```

使用yum search搜索高版本报错

```
yum search autoconfig
```

![image-20200725165758186](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200725165758186.png)



> 解决方式：替换阿里云仓库
>
> 阿里云仓库：mirrors.aliyun.com -> epel -> **epel(RHEL 6)**
>
> ![image-20200725170119244](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200725170119244.png)
>
> 
>
> ```
> cd /etc/yum.repos.d/
> wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
> yum clear all
> ```

此时再执行

```
yum search autoconfig
```

![image-20200725170643872](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200725170643872.png)

安装autoconf268

```
yum install autoconf268
```

安装完

```
autoreconf268 -fvi //会多一个configure
./configure --enable-debug=full
make
src/nutcracker -h
```

/root/soft/twemproxy/twemproxy/scripts，nutcracker.init是twemproxy启动脚本文件，拷贝init文件

```
cp nutcracker.init /etc/init.d/twemproxy
```

进入etc/init.d/

```
cd /etc/init.d/
```

查看文件，此时文件是不可执行文件

![image-20200801100124973](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801100124973.png)

更改执行权限

```
chmod +x twemproxy
```

期望配置文件地址

![image-20200801100303108](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801100303108.png)

进入twemproxy config目录

```
/root/soft/twemproxy/twemproxy/conf
```

![image-20200727215500217](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200727215500217.png)

复制所有文件至etc/

```
cp ./* /etc/nutcracker
```

进入/root/soft/twemproxy/twemproxy/src，拷贝nutcracker（可执行程序）至bin，可以在任何位置执行程序nutcracker

```
 cp nutcracker /usr/bin
```

进入cd /etc/nutcracker/修改配置文件，习惯拷贝一份，防止改错

```
cd /etc/nutcracker/
// 备份配置文件
cp nutcracker.yml nutcracker.yml.bak
```

修改配置文件，配置文件的介绍可以看官网

![image-20200801092038827](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801092038827.png)

按住dG删除127.0.0.1:6379:1后所有内容

![image-20200801092832794](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801092832794.png)

yy+p，复制粘贴，修改文件

![image-20200801093022014](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801093022014.png)

Redis手动启动会以当前目录作为持久化目录，新建data 6379/6380目录，并手动启动Redis

```
mkdir data
cd data
mkdir 6379
cd 6379
redis-server --port 6379
redis-cli -p 6380 shutdown
```

启动twemproxy服务

```
service twemproxy start
```

![image-20200801100515455](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801100515455.png)



验证代理：

连接到代理服务器

```
redis-cli -p 22121
```

set一些值

```
127.0.0.1:22121> set k01 aaa
OK
127.0.0.1:22121> set k02 bbb
OK
127.0.0.1:22121> set k03 ccc
OK

6379机器
127.0.0.1:6379> keys *
1) "k02"
2) "k01"
3) "k03"
680机器
127.0.0.1:6380> keys *

```

由于分制，代理服务器有一些命令不可以使用

![image-20200801104646530](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801104646530.png)



附录：/root/soft/twemproxy/twemproxy/scripts，nutcracker.init是twemproxy启动脚本文件

```shell
#! /bin/sh
#
# chkconfig: - 55 45
# description:  Twitter's twemproxy nutcracker
# processname: nutcracker
# config: /etc/sysconfig/nutcracker

# Source function library.
. /etc/rc.d/init.d/functions

USER="nobody"
OPTIONS="-d -c /etc/nutcracker/nutcracker.yml"

if [ -f /etc/sysconfig/nutcracker ];then
    . /etc/sysconfig/nutcracker
fi

# Check that networking is up.
if [ "$NETWORKING" = "no" ]
then
    exit 0
fi

RETVAL=0
prog="nutcracker"

start () {
    echo -n $"Starting $prog: "
    #Test the config before start.
    daemon --user ${USER} ${prog} $OPTIONS -t >/dev/null  2>&1
    RETVAL=$?
    if [ $RETVAL -ne 0 ] ; then
        echo  "Config check fail! Please  use 'nutcracker -c /etc/nutcracker/nutcracker.yml' for detail."
        echo_failure;
        echo;
        exit 1
    fi

    daemon --user ${USER} ${prog} $OPTIONS
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/${prog}
}
stop () {
    echo -n $"Stopping $prog: "
    killproc ${prog}
    RETVAL=$?
    echo
    if [ $RETVAL -eq 0 ] ; then
        rm -f /var/lock/subsys/${prog}
    fi
}

restart () {
    stop
    start
}

if [ -f /etc/sysconfig/nutcracker ];then
    . /etc/sysconfig/nutcracker
fi

# Check that networking is up.
if [ "$NETWORKING" = "no" ]
then
    exit 0
fi

RETVAL=0
prog="nutcracker"

start () {
    echo -n $"Starting $prog: "
    #Test the config before start.
    daemon --user ${USER} ${prog} $OPTIONS -t >/dev/null  2>&1
    RETVAL=$?
    if [ $RETVAL -ne 0 ] ; then
        echo  "Config check fail! Please  use 'nutcracker -c /etc/nutcracker/nutcracker.yml' for detail."
        echo_failure;
        echo;
        exit 1
    fi

    daemon --user ${USER} ${prog} $OPTIONS
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/${prog}
}
stop () {
    echo -n $"Stopping $prog: "
    killproc ${prog}
    RETVAL=$?
    echo
    if [ $RETVAL -eq 0 ] ; then
        rm -f /var/lock/subsys/${prog}
    fi
}

restart () {
    stop
    start
}


# See how we were called.
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  status)
    status ${prog}
    ;;
  restart|reload)
    restart
    ;;
  condrestart)
    [ -f /var/lock/subsys/nutcracker ] && restart || :
    ;;
  *)
    echo $"Usage: $0 {start|stop|status|restart|reload|condrestart}"
    exit 1
esac

exit $?
```

### 3.2 predixy

predixy release地址：https://github.com/joyieldInc/predixy/releases

复制地址

![image-20200801105551525](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200801105551525.png)

soft目录下新建文件夹

```
mkdir predixy
cd predixy/
wget https://github.com/joyieldInc/predixy/releases/download/1.0.5/predixy-1.0.5-bin-amd64-linux.tar.gz
```


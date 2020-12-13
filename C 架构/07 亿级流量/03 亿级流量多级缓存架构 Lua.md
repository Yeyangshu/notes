# Lua + 

## 1 Lua简介

Lua 是由巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组于1993年开发的一种轻量、小巧的脚本语言，用标准 C 语言编写，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

官网：http://www.lua.org/

Redis 在 2.6 版本中推出了脚本功能，允许开发者将 Lua 语言编写的脚本传到 Redis 中执行。使用 Lua 脚本的优点有如下几点:

- 减少网络开销：本来需要多次请求的操作，可以一次请求完成，从而节约网络开销；

- 原子操作：Redis 会将整个脚本作为一个整体执行，中间不会执行其它命令；

- 复用：客户端发送的脚本会存储在 Redis 中，从而实现脚本的复用。

## 2 Redis 与 Lua 整合

### 2.1 Lua执行简单脚本

登录到 Redis 客户端后执行

#### 2.1.1 Hello World

```properties
eval   "return 1 + 1"    0
#命令    脚本        参数个数
```

![image-20201212222832522](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212222832522.png)

#### 2.1.2 带参数

```lua
EVAL script numkeys key [key ...] arg [arg ...] 
# numkeys 是key的个数，后边接着写key1 key2...  val1 val2....
```

案例

```lua
EVAL "local msg='hello world' return msg..KEYS[1]" 1 AAA BBB
"hello worldAAA"
```

表是基于1的，也就是说索引以数值1开始。所以在表中的第一个元素就是mytable[1]，第二个就是mytable[2]等等。 表中不能有nil值。如果一个操作表中有[1, nil, 3, 4]，那么结果将会是[1]——表将会在第一个nil截断。

![image-20201212223116573](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212223116573.png)

###  2.2 Lua执行独立脚本文件

#### 2.2.1 获取Redis中key的value

此时Redis中只有k1，没有k2

```lua
local key = KEYS[1]  
local list = redis.call("get", key);
return list;
```

![image-20201212225910905](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212225910905.png)

#### 2.2.2 读取Redis集合中的数据

```lua
local key=KEYS[1]
local list=redis.call("lrange", key, 0, -1);
return list;
```

![image-20201212230426300](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212230426300.png)

#### 2.2.3 统计点击次数

```lua
local msg='count:'
local count = redis.call("get","count")
if not count then
    redis.call("set","count",1)
end

redis.call("incr","count")

return msg..count+1
```

#### 2.2.2 执行lua脚本

##### 2.2.2.1 本地执行

在root目录下新建文件test.lua

```
redis-cli --eval test.lua key1 key2, arg1 arg2 arg3
```

##### 2.2.2.2 远程执行

这种方式只执行一次

```lua
redis-cli -h 192.168.2.161 -a密码 --eval /usr/local/luascript/test.lua name age , xiao6
```

### 2.3 Lua 与 Redis 交互

#### 2.3.1 Lua 脚本获取 EVAL & EVALSHA 命令的参数

通过 Lua 脚本的全局变量 KEYS 和 ARGV，能够访问 EVAL 和 EVALSHA 命令的 key [key ...] 参数和 arg [arg ...] 参数。

作为 Lua Table，能够将 KEYS 和 ARGV 作为一维数组使用，其下标从 1 开始。

#### 2.3.2 Lua 脚本内部执行 Redis 命令

Lua 脚本内部允许通过内置函数执行 Redis 命令：

```lua
redis.call()
redis.pcall()
```

两者非常相似，区别在于：

若 Redis 命令执行错误，

- `redis.call()`将错误抛出（即 EVAL & EVALSHA 执行出错）；

- `redis.pcall()` 将错误内容返回。

```lua
local msg='count:'  
local count = redis.call("get","count")  
if not count then          
    redis.call("set","count",1)  
end  redis.call("incr","count")  
return msg..count+1
```

### 2.4 Redis WATCH/MULTI/EXEC 与Lua

redis 原生支持监听、事务、批处理，那么还需要lua吗？

- 两者不存在竞争关系，而是增强关系，lua可以完成redis自身没有的功能

- 在lua中可以使用上一步的结果，也就是可以开发**后面操作依赖前面操作的执行结果的应用**，MULT中的命令都是独立操作

- redis可以编写模块增强功能，但是c语言写模块，太难了，lua简单的多
- 计算向移动数据
- 原子操作

**lua脚本尽量短小并且尽量保证同一事物写在一段脚本内，因为redis是单线程的，过长的执行会造成阻塞，影响服务器性能。**

### 2.5 Redis Lua 脚本管理

1. `script load` ：此命令用于将Lua脚本加载到Redis内存中  
2. `script exists  scripts exists sha1 [sha1 …]`  ：此命令用于判断sha1是否已经加载到Redis内存中  
3. `script flush` ：此命令用于清除Redis内存已经加载的所有Lua脚本，在执行script flush后，sha1不复存在  
4. `script kill` ：此命令用于杀掉正在执行的Lua脚本

![image-20201212231732941](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212231732941.png)

### 2.6 死锁

下面代码会进入死循环，导致redis无法接受其他命令。

```lua
eval "while true do end" 0 
```

另起客户端

```lua
127.0.0.1:6379> keys *
(error) BUSY Redis is busy running a script. You can only call SCRIPT KILL or SHUTDOWN NOSAVE.
```

但是可以接受 `SCRIPT KILL` or `SHUTDOWN NOSAVE` 两个命令

`SHUTDOWN NOSAVE` 不会进行持久化的操作

`SCRIPT KILL` 可以杀死正在执行的进程

### 2.7 生产环境下部署

#### 2.7.1 加载到redis

```lua
redis-cli script load "$(cat test.lua)"
```

得到sha1值，执行

```lua
redis-cli evalsha "7a2054836e94e19da22c13f160bd987fbc9ef146" 0
```

![image-20201212231732941](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201212231732941.png)

## 3 Openresty Nginx + Lua 

Nginx是一个主进程配合多个工作进程的工作模式，每个进程由单个线程来处理多个连接。

在生产环境中，我们往往会把cpu内核直接绑定到工作进程上，从而提升性能。

官方网站：https://openresty.org/cn/

### 3.1 安装

#### 3.1.1 预编译安装

以CentOS举例 其他系统参照：http://openresty.org/cn/linux-packages.html

你可以在你的 CentOS 系统中添加 openresty 仓库，这样就可以便于未来安装或更新我们的软件包（通过 yum update 命令）。运行下面的命令就可以添加我们的仓库：

- yum install yum-utils

- yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

  然后就可以像下面这样安装软件包，比如 openresty：

- yum install openresty

如果你想安装命令行工具 resty，那么可以像下面这样安装 openresty-resty 包：

- sudo yum install openresty-resty

#### 3.1.2 源码编译安装

#### 3.1.3 下载

http://openresty.org/cn/download.html

```shell
# 解压
cp -r openresty-1.19.3.1 /opt/soft/openresty

# 官方安装教程 http://openresty.org/cn/installation.html
tar -xzvf openresty-VERSION.tar.gz
# 示例中的 VERSION替换成 OpenResty的版本号, 比如 1.11.2.1。
cd openresty-VERSION/
./configure
make
sudo make install
```

进入 /opt/soft/openresty目录

`./configure`

然后在进入 `openresty-VERSION/ `目录, 然后输入以下命令配置:

 `./configure`

默认, `--prefix=/usr/local/openresty` 程序会被安装到`/usr/local/openresty`目录。

依赖 `gcc openssl-devel pcre-devel zlib-devel`

安装：`yum install gcc openssl-devel pcre-devel zlib-devel postgresql-devel`

您可以指定各种选项，比如

```properties
./configure --prefix=/opt/openresty \

            --with-luajit \

            --without-http_redis2_module \

            --with-http_iconv_module \

            --with-http_postgres_module
```

试着使用 `./configure --help` 查看更多的选项。

`make && make install`

#### 3.1.4 服务命令

##### 3.1.4.1 启动

`Service openresty start`

##### 3.1.4.2 停止

`Service openresty stop`

##### 3.1.4.3 检查配置文件是否正确

`Nginx -t`

##### 3.1.4.4 重新加载配置文件

`Service openresty reload`

##### 3.1.4.5 查看已安装模块和版本号

`Nginx -V`

### 3.2 测试lua脚本

```nginx
# 在 nginx.conf 中写入
location /lua {
    default_type text/html;
    content_by_lua '
        ngx.say("<p>Hello, World!</p>")
        ';
}
```

配置文件

![image-20201213001448919](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213001448919.png)

浏览器访问`http://127.0.0.1/lua`

![image-20201213001346809](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213001346809.png)

### 3.3 lua-nginx-module

ngx_lua_module 是一个 Nginx http 模块，它把 lua 解析器内嵌到 Nginx，用来解析并执行 lua 语言编写的网页后台脚本。

#### 3.3.1 创建配置文件lua.conf

```nginx
server {
    listen       80;
    server_name  localhost;

    location /lua {
        default_type text/html;
        content_by_lua_file conf/lua/hello.lua;
    }
}
```

#### 3.3.2 在Nginx.conf下引入Lua配置

`include       lua.conf;`

![image-20201213001718382](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213001718382.png)

#### 3.3.3 创建外部Lua脚本

`conf/lua/hello.lua`

内容：

`ngx.say("<p>Hello, World!</p>")`

访问`http://127.0.0.1/lua`

![image-20201213001913811](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213001913811.png)

#### 3.3.4 获取Nginx uri中的单一变量

获取`http://127.0.0.1/lua?a=100`中100的值

第一种方式直接在nginx.conf配置文件中获取请求url中参数a的值 ：

 ```nginx
location /nginx_var {
    default_type text/html;
    content_by_lua_block {
        ngx.say(ngx.var.arg_a)
    }
}
 ```

第二种方式：在脚本文件test.lua中获取：

```lua
ngx.say(ngx.var.arg_a)
```

#### 3.3.5 获取Nginx uri中的所有变量

```lua
local uri_args = ngx.req.get_uri_args()  

for k, v in pairs(uri_args) do  
    if type(v) == "table" then  
        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  
    else  
        ngx.say(k, ": ", v, "<br/>")  
    end  
end
```

![image-20201213002808877](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213002808877.png)

请求`http://127.0.0.1/lua?a=100&b=200`

![image-20201213002721375](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213002721375.png)

#### 3.3.6 获取Nginx请求头信息

```lua
local headers = ngx.req.get_headers()                         

ngx.say("Host : ", headers["Host"], "<br/>")  
ngx.say("user-agent : ", headers["user-agent"], "<br/>")  
ngx.say("user-agent : ", headers.user_agent, "<br/>")

for k,v in pairs(headers) do  
    if type(v) == "table" then  
        ngx.say(k, " : ", table.concat(v, ","), "<br/>")  
    else  
        ngx.say(k, " : ", v, "<br/>")  
    end  
end  
```

![image-20201213002843383](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213002843383.png)

#### 3.3.7 获取post请求参数

```lua
ngx.req.read_body()  

ngx.say("post args begin", "<br/>")  

local post_args = ngx.req.get_post_args()  

for k, v in pairs(post_args) do  
    if type(v) == "table" then  
        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  
    else  
        ngx.say(k, ": ", v, "<br/>")  
    end  
end
```

![image-20201213003112480](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003112480.png)

#### 3.3.8 http协议版本

```lua
ngx.say("ngx.req.http_version : ", ngx.req.http_version(), "<br/>")
```

![image-20201213003152348](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003152348.png)

#### 3.3.9 请求方法

```lua
ngx.say("ngx.req.get_method : ", ngx.req.get_method(), "<br/>")  
```

![image-20201213003216067](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003216067.png)

#### 3.3.10 原始的请求头内容  

```lua
ngx.say("ngx.req.raw_header : ",  ngx.req.raw_header(), "<br/>")  
```

![image-20201213003304893](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003304893.png)

#### 3.3.11 body内容体  

```lua
ngx.say("ngx.req.get_body_data() : ", ngx.req.get_body_data(), "<br/>")
```

### 3.4 Nginx缓存

#### 3.4.1 Nginx全局共享内存缓存

`lua_shared_dict`设置一块共享内存区域，可以被各个worker共享，写在HTTP模块中

```nginx
lua_shared_dict shared_data 1m;
```

![image-20201213003806203](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003806203.png)

Lua脚本

```lua
# Lua脚本
local shared_data = ngx.shared.shared_data  
local i = shared_data:get("i")  

if not i then  
    i = 1  
    shared_data:set("i", i)  
    ngx.say("lazy set i ", i, "<br/>")  
end  

i = shared_data:incr("i", 1)  
ngx.say("i=", i, "<br/>")
```

![image-20201213003900243](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213003900243.png)

#### 3.4.2 lua-resty-lrucache

Lua 实现的一个简单的 LRU 缓存，适合在 Lua 空间里直接缓存较为复杂的 Lua 数据结构：

它相比 ngx_lua 共享内存字典可以省去较昂贵的序列化操作，相比 memcached 这样的外部服务又能省去较昂贵的 socket 操作

lrucache 有两种实现

- resty.lrucache
  - 适合用来缓存命中率高或读操作远远大于写操作的缓存业务
- resty.lrucache.pureffi
  - 适合用来缓存命中率低或需要对key进行频繁增、删操作的缓存业务

```lua
local lrucache = require "resty.lrucache"

local c, err = lrucache.new(200)

	c:set("dog", 32)
    c:set("cat", 56)
    ngx.say("dog: ", c:get("dog"))
    ngx.say("cat: ", c:get("cat"))

    c:set("dog", { age = 10 }, 0.1)  -- expire in 0.1 sec
    c:delete("dog")
```

https://github.com/openresty/lua-resty-lrucache#name

![image-20201213004040247](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213004040247.png)



##### 3.4.2.1 实例

```lua
local mycache = require("mycache")  
local count = mycache.get("count") or 0  
count = count + 1  
mycache.set("count", count, 10 * 60 * 60) --10分钟  
ngx.say(mycache.get("count"))     
```

**mycache.lua**

```lua
local lrucache = require("resty.lrucache")  
--创建缓存实例，并指定最多缓存多少条目  
local cache, err = lrucache.new(200)  
if not cache then  
   ngx.log(ngx.ERR, "create cache error : ", err)  
end  
  
local function set(key, value, ttlInSeconds)  
    cache:set(key, value, ttlInSeconds)  
end  
  
local function get(key)  
    return cache:get(key)  
end  
  
local _M = {  
  set = set,  
  get = get  
}  
return _M
```

##### 3.4.2.2 tmpfs

在Linux系统内存中的虚拟磁盘映射，可以理解为使用物理内存当做磁盘，利用这种文件系统，可以有效提高在高并发场景下的磁盘读写，但是重启后数据会丢失。

###### 3.4.2.2.1 查看tmpfs命令：



![](C:\Study\InternetArchitect-master\12-1 亿级流量copy\images\1569584708275.png)

###### 3.4.2.2.2 系统默认开启，大小约为物理内存一半



###### 3.4.2.2.3 查看物理内存利用情况

![image-20201213135535690](X:\Users\11077\AppData\Roaming\Typora\typora-user-images\image-20201213135535690.png)

tmpfs没有占用内存空间，只有在写入数据的时候才会占用实际的物理内存

###### 3.4.2.2.4 调整大小

临时修改方式如下，立即生效但重启后会恢复

![1569584781944](C:\Study\InternetArchitect-master\12-1 亿级流量copy\images\1569584781944.png)

###### 3.4.2.2.5 永久修改 /etc/fstab文件

![1569584800821](C:\Study\InternetArchitect-master\12-1 亿级流量copy\images\1569584800821.png)

#### 3.4.3 http_proxy 本地磁盘缓存

```nginx
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
     set $upstream http://ip:port
          location / {
                   proxy_cache my_cache;
                   proxy_pass $upstream;
             }
}
```

` /path/to/cache`  #本地路径，用来设置Nginx缓存资源的存放地址

`  levels `         #默认所有缓存文件都放在同一个`/path/to/cache`下，但是会影响缓存的性能，因此通常会在`/path/to/cache`下面建立子目录用来分别存放不同的文件。假设`levels=1:2`，Nginx为将要缓存的资源生成的`key`为`f4cd0fbc769e94925ec5540b6a4136d0`，那么`key`的最后一位0，以及倒数第2-3位6d作为两级的子目录，也就是该资源最终会被缓存到`/path/to/cache/0/6d`目录中

`key_zone`        #在共享内存中设置一块存储区域来存放缓存的`key`和`metadata`（类似使用次数），这样nginx可以快速判断一个request是否命中或者未命中缓存，1m可以存储8000个key，10m可以存储80000个key

`max_size`        #最大cache空间，如果不指定，会使用掉所有disk space，当达到配额后，会删除最少使用的cache文件

` inactive`        #未被访问文件在缓存中保留时间，本配置中如果60分钟未被访问则不论状态是否为expired，缓存控制程序会删掉文件。inactive默认是10分钟。需要注意的是，inactive和expired配置项的含义是不同的，expired只是缓存过期，但不会被删除，inactive是删除指定时间内未被访问的缓存文件

` use_temp_path`   #如果为off，则nginx会将缓存文件直接写入指定的cache文件中，而不是使用temp_path存储，official建议为off，避免文件在不同文件系统中不必要的拷贝

`proxy_cache`     #启用proxy cache，并指定key_zone。另外，如果proxy_cache off表示关闭掉缓存。

### 3.5 lua-resty-redis访问Redis（推荐）

样例

```lua
# 引入/lualib/resty/resty.redis，没有会报错
local redis = require "resty.redis"
local red = redis:new()
red:set_timeout(1000)
local ok, err = red.connect("127.0.0.1", "6379")
	if not ok then
		ngx.say("fail to connect：", err)
		return
	end
ok, err = red:set("dog", "an animal")
	if not ok then
		ngx.say("fail to set dog：", err)
		return
	end
```

#### 3.5.1 常用方法

```lua
local res, err = red:get("key")
local res, err = red:lrange("nokey", 0, 1)
ngx.say("res:",cjson.encode(res))
```

#### 3.5.2 创建Redis连接

```lua
red, err = redis:new()
ok, err = red:connect(host, port, options_table?)
```

#### 3.5.3 timeout

```lua
red:set_timeout(time)
```

#### 3.5.4 keepalive

```lua
red:set_keepalive(max_idle_timeout, pool_size)
```

#### 3.5.5 close

```
ok, err = red:close()
```

#### 3.5.6 pipeline

```nginx
red:init_pipeline()
results, err = red:commit_pipeline()
```

#### 3.5.7 认证

```lua
local res, err = red:auth("foobared")
if not res then
    ngx.say("failed to authenticate: ", err)
    return
end
```

#### 3.5.8 redis-cluster支持

https://github.com/steve0511/resty-redis-cluster

### 3.6 redis2-nginx-module （不推荐）

redis2-nginx-module是一个支持 Redis 2.0 协议的 Nginx upstream 模块，它可以让 Nginx 以非阻塞方式直接防问远方的 Redis 服务，同时支持 TCP 协议和 Unix Domain Socket 模式，并且可以启用强大的 Redis 连接池功能。

https://github.com/openresty/redis2-nginx-module

#### 3.6.1 test

```nginx
location = /foo {
    default_type text/html;
    # Redis登录密码
    redis2_query auth 123123;
    set $value 'first';
    redis2_query set one $value;
    redis2_pass 192.168.199.161:6379;
}
```

#### 3.6.2 get

```nginx
location = /get {

default_type text/html;
     redis2_pass 192.168.199.161:6379;
     redis2_query auth 123123;
     set_unescape_uri $key $arg_key;  # this requires ngx_set_misc
     redis2_query get $key;
}
```

#### 3.6.3 set

```nginx
# GET /set?key=one&val=first%20value
location = /set {
    default_type text/html;
    redis2_pass 192.168.199.161:6379;
    redis2_query auth 123123;

    set_unescape_uri $key $arg_key;  # this requires ngx_set_misc
    set_unescape_uri $val $arg_val;  # this requires ngx_set_misc
    redis2_query set $key $val;
}
```

#### 3.6.4 pipeline

```nginx
set $value 'first';
redis2_query set one $value;
redis2_query get one;
redis2_query set one two;
redis2_query get one;
redis2_query del key1;
```

#### 3.6.5 list

```lua
redis2_query lpush key1 C;
redis2_query lpush key1 B;
redis2_query lpush key1 A;
redis2_query lrange key1 0 -1;
```

#### 3.6.6 集群

```nginx
upstream redis_cluster {

    server 192.168.199.161:6379;

    server 192.168.199.161:6379;

}

location = /redis {

    default_type text/html;

    redis2_next_upstream error timeout invalid_response;

    redis2_query get foo;

    redis2_pass redis_cluster;
}
```

## 4 URL一致性哈希负载均衡

有针对性的对url进行一致性 hash 定向负载到后端 Nginx 

提高Nginx缓存系统命中率

### 4.1 nginx url_hash

Nginx第三方模块，在转发请求时如果后端服务器宕机，会导致503错误

> 老师：难扩展

### 4.2 lua-resty-http

#### 4.2.1 GitHub主页

https://github.com/ledgetech/lua-resty-http

将两个文件导入到`lualib\resty`文件夹内

#### 4.2.2 安装组件

wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua  

wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua 

```lua
local http = require("resty.http")  
local httpc = http.new()  
local resp, err = httpc:request_uri("http://www.sogou.com", {  
        method = "GET",
        # uri
        path = "/sogou?query=resty.http",  
        headers = {  
            ["User-Agent"] = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"  
        }  
    })

if not resp then  
    ngx.say("request error :", err)  
    return  
end  

ngx.status = resp.status  

for k, v in pairs(resp.headers) do  
    if k ~= "Transfer-Encoding" and k ~= "Connection" then  
        ngx.header[k] = v  
    end  
end  

ngx.say(resp.body)  
httpc:close()
```

http模块加入

```nginx
resolver 8.8.8.8;
```

server加入（也可以不加）

```nginx
location = /lb {
    default_type text/html;
    content_by_lua_file lua/script/lb.lua;
}
```

浏览器访问：`http://127.0.0.1/lb`，发送请求，跳转至搜狗

![image-20201213142032915](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213142032915.png)

#### 4.2.3 lua-resty-http实现一致性hash负载均衡

自定义扩展：负载均衡等等功能

```lua
-- 加载模块
local http = require("resty.http")  
local httpc = http.new()  

-- 服务器列表
local hosts = {"192.168.150.111","192.168.150.112"}

local item_id= ngx.var.arg_id

-- hash
local id_hash = ngx.crc32_long(item_id)
local index = (id_hash % 2) +1

local resp, err = httpc:request_uri("http://"..hosts[index], {  
    method = "GET",  
    path = "/sogou?query=resty.http",  
    headers = {  
        ["User-Agent"] = "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36"  
    }  
})  

if not resp then  
    ngx.say("request error :", err)  
    return  
end  
  
 
ngx.say(resp.body)  
httpc:close()
```

## 5 模板实时渲染 lua-resty-template

https://github.com/bungle/lua-resty-template

如果学习过JavaEE中的servlet和JSP的话，应该知道JSP模板最终会被翻译成Servlet来执行；

而lua-resty-template模板引擎可以认为是JSP，其最终会被翻译成Lua代码，然后通过ngx.print输出。   

> 从内存加载模板和数据

lua-resty-template大体内容有： 

- 模板位置：从哪里查找模板； 

- 变量输出/转义：变量值输出； 

- 代码片段：执行代码片段，完成如if/else、for等复杂逻辑，调用对象函数/方法； 

- 注释：解释代码片段含义； 

- include：包含另一个模板片段； 

- 其他：lua-resty-template还提供了不需要解析片段、简单布局、可复用的代码块、宏指令等支持。

基础语法

- {(include_file)}：包含另一个模板文件；

- {* var *}：变量输出；

- {{ var }}：变量转义输出；

- {% code %}：代码片段；

- {# comment #}：注释；

- {-raw-}：中间的内容不会解析，作为纯文本输出；

### 5.1 lua代码热加载

在http模块中加入

```nginx
lua_code_cache off;
```

reload后Nginx会提示影响性能，记得在生产环境中关掉。

![image-20201213151604683](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213151604683.png)

### 5.2 测试

#### 5.2.1 初始化

```lua
-- 两种方式
-- 1 Using template.new
local template = require "resty.template"
local view = template.new "view.html"
view.message = "Hello, World!"
view:render()

-- 2 Using template.render
-- template.render("view.html", { message = "Hel11lo, Worl1d!" })

```

![image-20201213200030125](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213200030125.png)

#### 5.2.2 执行函数，得到渲染之后的内容

```
local func = template.compile("view.html")  
local content = func(context)  
ngx.say("xx:",content) 
```

#### 5.2.3 resty.template.html

```lua
local template = require("resty.template")
local html = require "resty.template.html"

template.render([[
<ul>
{% for _, person in ipairs(context) do %}
    {*html.li(person.name)*} --
{% end %}
</ul>
<table>
{% for _, person in ipairs(context) do %}
    <tr data-sort="{{(person.name or ""):lower()}}">
        {*html.td{ id = person.id }(person.name)*}
    </tr>
{% end %}
</table>]], {
    { id = 1, name = "Emma"},
    { id = 2, name = "James" },
    { id = 3, name = "Nicholas" },
    { id = 4 }
})
```

![image-20201213200620046](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201213200620046.png)

#### 5.2.4 模板内容

模板文件放在html文件夹下

```html
<!DOCTYPE html>
<html>
<body>
  <h1>{{message}}</h1>
</body>
</html>

```

### 5.3 多值传入

```lua
template.caching(false)
local template = require("resty.template")
local context = {
    name = "lucy",
    age = 50,
}
template.render("view.html", context)
```

模板内容

```lua
<!DOCTYPE html>
<html>
<body>
  <h1>name:{{name}}</h1>
  <h1>age:{{age}}</h1>
</body>
</html>

```

### 5.4 模板管理与缓存

模板缓存：默认开启，开发环境可以手动关闭

```template.caching(true)```

模板文件需要业务系统更新与维护，当模板文件更新后，可以通过模板版本号或消息通知Openresty清空缓存重载模板到内存中

`template.cache = {}`

### 5.5 完整页面

```lua
local template = require("resty.template")
template.caching(false)
local context = {
    title = "测试",
    name = "lucy",
    description = "<script>alert(1);</script>",
    age = 40,
    hobby = {"电影", "音乐", "阅读"},
    score = {语文 = 90, 数学 = 80, 英语 = 70},
    score2 = {
        {name = "语文", score = 90},
        {name = "数学", score = 80},
        {name = "英语", score = 70},
    }
}

template.render("view.html", context)
```

模板

```html
{(header.html)}  
   <body>  
      {# 不转义变量输出 #}  
      姓名：{* string.upper(name) *}<br/>  
      {# 转义变量输出 #}  
      简介：{{description}}
           简介：{* description *}<br/>  
      {# 可以做一些运算 #}  
      年龄: {* age + 10 *}<br/>  
      {# 循环输出 #}  
      爱好：  
      {% for i, v in ipairs(hobby) do %}  
         {% if v == '电影' then  %} - xxoo
            
              {%else%}  - {* v *} 
{% end %}  
         
      {% end %}<br/>  
  
      成绩：  
      {% local i = 1; %}  
      {% for k, v in pairs(score) do %}  
         {% if i > 1 then %}，{% end %}  
         {* k *} = {* v *}  
         {% i = i + 1 %}  
      {% end %}<br/>  
      成绩2：  
      {% for i = 1, #score2 do local t = score2[i] %}  
         {% if i > 1 then %}，{% end %}  
          {* t.name *} = {* t.score *}  
      {% end %}<br/>  
      {# 中间内容不解析 #}  
      {-raw-}{(file)}{-raw-}  
{(footer.html)}  
```

### 5.6 layout 布局统一风格

使用模板内容嵌套可以实现全站风格同一布局

#### 5.6.1 lua

`local template = require "resty.template"`

1

```
local layout   = template.new "layout.html"

layout.title   = "Testing lua-resty-template"

layout.view    = template.compile "view.html" { message = "Hello, World!" }

layout:render()
```

2

```
template.render("layout.html", {

  title = "Testing lua-resty-template",

  msg = "type=2",

  view  = template.compile "view.html" { message = "Hello, World!" }

})
```

3

此方式重名变量值会被覆盖

```
local view     = template.new("view.html", "layout.html")

view.title     = "Testing lua-resty-template"

view.msg = "type=3"

view.message   = "Hello, World!"

view:render()
```

4

可以区分一下

```
local layout   = template.new "layout.html"

layout.title   = "Testing lua-resty-template"

layout.msg = "type=4"

local view     = template.new("view.html", layout)

view.message   = "Hello, World!"

view:render()
```

#### 5.6.2 layout.html

```html
<!DOCTYPE html>
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <h1>layout</h1>
    <body>
        {*view*}
    </body>
</html>
```



#### 5.6.3 viewhtml

`msg:{{message}}`

#### 5.6.4 多级嵌套

lua

```lua
local view     = template.new("view.html", "layout.html")
view.title     = "Testing lua-resty-template"
view.message   = "Hello, World!"
view:render()
view.html
{% layout="section.html" %}
```

html

```html
<h1>msg:{{message}}</h1>
section.html
<div
     id="section">
    {*view*} - sss
</div>
layout.html
<!DOCTYPE html>
<html>
    <head>
        <title>{{title}}</title>
    </head>

    <h1>layout {{msg}}</h1>
    <body>
        {*view*}
    </body>
</html>
```

## 6 IDE Lua 脚本调试

### 6.1 EmmyLua插件

https://github.com/EmmyLua/IntelliJ-EmmyLua

https://emmylua.github.io/zh_CN/

### 6.2 LDT 基于eclipse

https://www.eclipse.org/ldt/

## 7 限流

### 7.1 限流算法

#### 7.1.1 漏桶算法                     

#### 7.1.2 令牌桶算法

#### 7.1.3 计数器

常见的连接池、线程池等简单粗暴的按照数量限流的方式

### 7.2 Tomcat限流

server.xml配置文件中的Connector节点

```nginx
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" 
maxConnections="800" acceptCount="500" maxThreads="400" />
```

- maxThreads：tomcat能并发处理的最大线程数

- acceptCount：当tomcat起动的线程数达到最大时，接受排队的请求个数，默认值为100 

- maxConnections：瞬时最大连接数，超出会排队等待

## 8 Lua 开源项目

### 8.1 WAF

https://github.com/unixhot/waf

https://github.com/loveshell/ngx_lua_waf

- 防止 SQL 注入，本地包含，部分溢出，fuzzing 测试，XSS/SSRF 等 Web 攻击

- 防止 Apache Bench 之类压力测试工具的攻击

- 屏蔽常见的扫描黑客工具，扫描器

- 屏蔽图片附件类目录执行权限、防止 webshell 上传

- 支持 IP 白名单和黑名单功能，直接将黑名单的 IP 访问拒绝

- 支持 URL 白名单，将不需要过滤的 URL 进行定义

- 支持 User-Agent 的过滤、支持 CC 攻击防护、限制单个 URL 指定时间的访问次数

- 支持支持 Cookie 过滤，URL 与 URL 参数过滤

- 支持日志记录，将所有拒绝的操作，记录到日志中去

### 8.2 Kong 基于Openresty的流量网关

https://konghq.com/

https://github.com/kong/kong

Kong 基于 OpenResty，是一个云原生、快速、可扩展、分布式的微服务抽象层（Microservice Abstraction Layer），也叫 API 网关（API Gateway），在 Service Mesh 里也叫 API 中间件（API Middleware）。

Kong 开源于 2015 年，核心价值在于高性能和扩展性。从全球 5000 强的组织统计数据来看，Kong 是现在依然在维护的，在生产环境使用最广泛的 API 网关。

Kong 宣称自己是世界上最流行的开源微服务 API 网关（The World’s Most Popular Open Source Microservice API Gateway）。

核心优势：

- 可扩展：可以方便的通过添加节点水平扩展，这意味着可以在很低的延迟下支持很大的系统负载。

- 模块化：可以通过添加新的插件来扩展 Kong 的能力，这些插件可以通过 RESTful Admin API 来安装和配置。

- 在任何基础架构上运行：Kong 可以在任何地方都能运行，比如在云或混合环境中部署 Kong，单个或全球的数据中心。

### 8.3 ABTestingGateway

https://github.com/CNSRE/ABTestingGateway

ABTestingGateway 是一个可以动态设置分流策略的网关，关注与灰度发布相关领域，基于 Nginx 和 ngx-lua 开发，使用 Redis 作为分流策略数据库，可以实现动态调度功能。

ABTestingGateway 是新浪微博内部的动态路由系统 dygateway 的一部分，目前已经开源。在以往的基于 Nginx 实现的灰度系统中，分流逻辑往往通过 rewrite 阶段的 if 和 rewrite 指令等实现，优点是性能较高，缺点是功能受限、容易出错，以及转发规则固定，只能静态分流。ABTestingGateway 则采用 ngx-lua，通过启用 lua-shared-dict 和 lua-resty-lock 作为系统缓存和缓存锁，系统获得了较为接近原生 Nginx 转发的性能。

- 支持多种分流方式，目前包括 iprange、uidrange、uid 尾数和指定uid分流

- 支持多级分流，动态设置分流策略，即时生效，无需重启

- 可扩展性，提供了开发框架，开发者可以灵活添加新的分流方式，实现二次开发

- 高性能，压测数据接近原生 Nginx 转发

-   灰度系统配置写在 Nginx 配置文件中，方便管理员配置

-   适用于多种场景：灰度发布、AB 测试和负载均衡等
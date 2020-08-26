# 服务器参数设置

## 1 general

1. datadir=/var/lib/mysql

   数据文件存放的目录

2. socket=var/lib/mysql/mysql.sock

   mysql.socket表示server和client在同一台服务器，并且使用localhost进行连接，就会使用socket进行连接

3. pid_file=/var/lib/mysql/mysql.pid

   存储mysql的pid

4. port=3306

   mysql服务的端口号

5. default_storge_engine=InnoDB

   mysql存储引擎

6. skip-grant-tables

   当忘记mysql的用户名密码的时候，可以在mysql配置文件中配置该参数，跳过权限表验证，不需要密码即可登录mysql

## 2 character

1. character_set_client

   客户端数据的字符集

2. character_set_connection

   mysql处理客户端发来的信息时，会把这些数据转换成连接的字符集格式

3. character_set_results

   mysql发送给客户端的结果集所用的字符集

4. character_set_database

   数据库默认的字符集

5. character_set_server

   mysql server的默认字符集

## 3 connection

1. max_connections

   mysql的最大连接数，如果数据库的并发连接请求比较大，应该调高该值

2. max_user_connections

   限制每个用户的连接个数

3. back_log

   mysql能够暂存的连接数量，当mysql的线程在一个很短时间内得到非常多的连接请求时，就会起作用，如果mysql的连接数量达到max_connections时，新的请求会被存储在堆栈中，以等待某一个连接释放资源，如果等待连接的数量超过back_log,则不再接受连接资源

4. wait_timeout

   mysql在关闭一个非交互的连接之前需要等待的时长

5. interactive_timeout

   关闭一个交互连接之前需要等待的秒数

## 4 log

1. log_error
   	指定错误日志文件名称，用于记录当mysqld启动和停止时，以及服务器在运行中发生任何严重错误时的相关信息

2. log_bin

   指定二进制日志文件名称，用于记录对数据造成更改的所有查询语句（建议开启）
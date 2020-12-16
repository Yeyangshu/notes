2. # 服务器参数设置

   ## 1 general
   
   1. datadir=/var/lib/MySQL
   
      数据文件存放的目录
   
   2. socket=var/lib/MySQL/MySQL.sock
   
      MySQL.socket表示server和client在同一台服务器，并且使用localhost进行连接，就会使用socket进行连接
   
   3. pid_file=/var/lib/MySQL/MySQL.pid
   
      存储MySQL的pid
   
   4. port=3306
   
      MySQL服务的端口号
   
   5. default_storge_engine=InnoDB
   
      MySQL存储引擎
   
   6. skip-grant-tables
   
      当忘记MySQL的用户名密码的时候，可以在MySQL配置文件中配置该参数，跳过权限表验证，不需要密码即可登录MySQL
   
   ## 2 character
   
   1. character_set_client
   
      客户端数据的字符集
   
      ```sql
      mysql> SHOW VARIABLES LIKE '%character_set_client%';
      -- utf8mb4
      ```
   
   2. character_set_connection
   
      MySQL处理客户端发来的信息时，会把这些数据转换成连接的字符集格式
   
      ```sql
      mysql> SHOW VARIABLES LIKE '%character_set_connection%';
      -- utf8mb4
      ```
   
   3. character_set_results
   
      MySQL发送给客户端的结果集所用的字符集
   
      ```sql
      mysql> SHOW VARIABLES LIKE '%character_set_results%';
      -- utf8mb4
      ```
   
   4. character_set_database
   
      数据库默认的字符集
   
      ```sql
      mysql> SHOW VARIABLES LIKE '%character_set_database%';
      +------------------------+-------+
      | Variable_name          | Value |
      +------------------------+-------+
      | character_set_database | utf8  |
      +------------------------+-------+
      ```
   
   5. character_set_server
   
      MySQL server的默认字符集
   
      ```sql
      mysql> SHOW VARIABLES LIKE '%character_set_server%';
      +----------------------+-------+
      | Variable_name        | Value |
      +----------------------+-------+
      | character_set_server | utf8  |
      +----------------------+-------+
      1 row in set (0.01 sec)
      ```
   
   ## 3 connection
   
   1. max_connections
   
      MySQL的最大连接数，如果数据库的并发连接请求比较大，应该调高该值
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'max_connections%';
      +-----------------+-------+
      | Variable_name   | Value |
      +-----------------+-------+
      | max_connections | 80000 |
      +-----------------+-------+
      1 row in set (0.01 sec)
      ```
   
   2. max_user_connections
   
      限制每个用户的连接个数
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'max_user_connections';
      +----------------------+-------+
      | Variable_name        | Value |
      +----------------------+-------+
      | max_user_connections | 8000  |
      +----------------------+-------+
      1 row in set (0.01 sec)
      ```
   
   3. back_log
   
      MySQL能够暂存的连接数量，当MySQL的线程在一个很短时间内得到非常多的连接请求时，就会起作用，如果MySQL的连接数量达到max_connections时，新的请求会被存储在堆栈中，以等待某一个连接释放资源，如果等待连接的数量超过back_log，则不再接受连接资源
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'back_log';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | back_log      | 1024  |
      +---------------+-------+
      1 row in set (0.01 sec)
      ```
   
   4. wait_timeout
   
      MySQL在关闭一个非交互的连接之前需要等待的时长
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'wait_timeout';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | wait_timeout  | 28800 |
      +---------------+-------+
      1 row in set (0.01 sec)
      ```
   
   5. interactive_timeout
   
      关闭一个交互连接之前需要等待的秒数
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'interactive_timeout';
      +---------------------+-------+
      | Variable_name       | Value |
      +---------------------+-------+
      | interactive_timeout | 28800 |
      +---------------------+-------+
      ```
   
   ## 4 log
   
   1. log_error
   
      指定错误日志文件名称，用于记录当Mysqld启动和停止时，以及服务器在运行中发生任何严重错误时的相关信息
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'log_error';
      +---------------+---------------------------+
      | Variable_name | Value                     |
      +---------------+---------------------------+
      | log_error     | /u01/my3306/log/alert.log |
      +---------------+---------------------------+
      1 row in set (0.01 sec)
      ```
   
   2. log_bin
   
      指定二进制日志文件名称，用于记录对数据造成更改的所有查询语句（建议开启）
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'log_bin';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | log_bin       | ON    |
      +---------------+-------+
      1 row in set (0.01 sec)
      ```
   
   3. binlog_do_db
   
      指定将更新记录到二进制日志的数据库，其他所有没有显式指定的数据库更新将忽略，不记录在日志中
   
   4. inlog_ignore_db
   
      指定不将更新记录到二进制日志的数据库
   
   5. sync_binlog
   
      指定多少次写日志后同步磁盘
   
   6. general_log
   
      是否开启查询日志记录
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'general_log';
      +---------------+-------+
      | Variable_name | Value |
      +---------------+-------+
      | general_log   | OFF   |
      +---------------+-------+
      1 row in set (0.01 sec)
      ```
   
   7. general_log_file
   
      指定查询日志文件名，用于记录所有的查询语句
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'general_log_file';
      +------------------+-----------------------------+
      | Variable_name    | Value                       |
      +------------------+-----------------------------+
      | general_log_file | /u01/my3306/log/general.log |
      +------------------+-----------------------------+
      1 row in set (0.03 sec)
      ```
   
   8. slow_query_log
   
      是否开启慢查询日志记录
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'slow_query_log';
      +----------------+-------+
      | Variable_name  | Value |
      +----------------+-------+
      | slow_query_log | ON    |
      +----------------+-------+
      1 row in set (0.01 sec)
      ```
   
   9. slow_query_log_file
   
      指定慢查询日志文件名称，用于记录耗时比较长的查询语句
   
      ```sql
      mysql> SHOW VARIABLES LIKE 'slow_query_log_file';
      +---------------------+--------------------------+
      | Variable_name       | Value                    |
      +---------------------+--------------------------+
      | slow_query_log_file | /u01/my3306/log/slow.log |
      +---------------------+--------------------------+
      1 row in set (0.02 sec)
      ```
   
   10. long_query_time
   
       设置慢查询的时间，超过这个时间的查询语句才会记录日志
   
       ```sql
       mysql> SHOW VARIABLES LIKE 'long_query_time';
       +-----------------+----------+
       | Variable_name   | Value    |
       +-----------------+----------+
       | long_query_time | 1.000000 |
       +-----------------+----------+
       1 row in set (0.01 sec)
       ```
   
   11. log_slow_admin_statements
   
       是否将管理语句写入慢查询日志
   
       ```sql
       mysql> SHOW VARIABLES LIKE 'log_slow_admin_statements';
       +---------------------------+-------+
       | Variable_name             | Value |
       +---------------------------+-------+
       | log_slow_admin_statements | ON    |
       +---------------------------+-------+
       1 row in set (0.01 sec)
       ```
   
   ## 5 cache
   
   ### 5.1 key_buffer_size
   
   索引缓存区的大小（只对myisam表起作用）
   
   ```sql
   mysql> SHOW VARIABLES LIKE 'key_buffer_size';
   +-----------------+---------+
   | Variable_name   | Value   |
   +-----------------+---------+
   | key_buffer_size | 8388608 |
   +-----------------+---------+
   1 row in set (0.01 sec)
   
   -- 默认8M
   ```
   
   ### 5.2 query cache
   
   #### 5.2.1 query_cache_size
   
   查询缓存的大小，未来版本被删除
   
   show status like '%Qcache%';查看缓存的相关属性
   
   - Qcache_free_blocks：缓存中相邻内存块的个数，如果值比较大，那么查询缓存中碎片比较多
   
   - Qcache_free_memory：查询缓存中剩余的内存大小
   
   - Qcache_hits：表示有多少此命中缓存
   
   - Qcache_inserts：表示多少次未命中而插入
   
   - Qcache_lowmen_prunes：多少条query因为内存不足而被移除cache
   
   - Qcache_queries_in_cache：当前cache中缓存的query数量
   
   - Qcache_total_blocks：当前cache中block的数量
   
   #### 5.2.2 query_cache_limit
   
   超出此大小的查询将不被缓存
   
   #### 5.2.3 query_cache_min_res_unit
   
   缓存块最小大小
   
   #### 5.2.4 query_cache_type
   
   缓存类型，决定缓存什么样的查询
   
   - 0表示禁用
   
   - 1表示将缓存所有结果，除非sql语句中使用sql_no_cache禁用查询缓存
   
   - 2表示只缓存select语句中通过sql_cache指定需要缓存的查询
   
   ### 5.3 sort_buffer_size
   
   每个需要排序的线程分配该大小的缓冲区
   
   ### 5.4 max_allowed_packet=32M
   
   限制server接受的数据包大小
   
   ### 5.5 join_buffer_size=2M
   
   表示关联缓存的大小
   
   ### 5.6 thread_cache_size
   
   1. Threads_cached：代表当前此时此刻线程缓存中有多少空闲线程
   
   2. Threads_connected：代表当前已建立连接的数量
   
   3. Threads_created：代表最近一次服务启动，已创建现成的数量，如果该值比较大，那么服务器会一直再创建线程
   
   4. Threads_running：代表当前激活的线程数
   
   ```sql
   mysql> show status like 'threads%';
   +-------------------------------+---------+
   | Variable_name                 | Value   |
   +-------------------------------+---------+
   | Threads_cached                | 9       |
   | Threads_connected             | 17302   |
   | Threads_created               | 1126730 |
   | Threads_running               | 23      |
   | Threads_running_in_async_mode | 23      |
   +-------------------------------+---------+
   5 rows in set (0.01 sec)
   ```
   
   ## 6 INNODB
   
   1. innodb_buffer_pool_size=
   
      该参数指定大小的内存来缓冲数据和索引，最大可以设置为物理内存的80%
   
   2. innodb_flush_log_at_trx_commit
   
      主要控制innodb将log buffer中的数据写入日志文件并flush磁盘的时间点，值分别为0，1，2
   
      > 基础班那张图
   
   3. innodb_thread_concurrency
   
      设置innodb线程的并发数，默认为0表示不受限制，如果要设置建议跟服务器的cpu核心数一致或者是cpu核心数的两倍
   
   4. innodb_log_buffer_size
   
      此参数确定日志文件所用的内存大小，以M为单位
   
   5. innodb_log_file_size
   
      此参数确定数据日志文件的大小，以M为单位
   
   6. innodb_log_files_in_group
   
      以循环方式将日志文件写到多个文件中
   
   7. read_buffer_size
   
      mysql读入缓冲区大小，对表进行顺序扫描的请求将分配到一个读入缓冲区
   
   8. read_rnd_buffer_size
   
      mysql随机读的缓冲区大小
   
   9. innodb_file_per_table
   
      此参数确定为每张表分配一个新的文件
   
   ```sql
   mysql> show status like '%innodb_%';
   +---------------------------------------+--------------------------------------------------+
   | Variable_name                         | Value                                            |
   +---------------------------------------+--------------------------------------------------+
   | Innodb_buffer_pool_dump_status        | Dumping of buffer pool not started               |
   | Innodb_buffer_pool_load_status        | Buffer pool(s) load completed at 201117 10:45:37 |
   | Innodb_buffer_pool_resize_status      |                                                  |
   | Innodb_buffer_pool_pages_data         | 637338                                           |
   | Innodb_buffer_pool_bytes_data         | 10442145792                                      |
   | Innodb_buffer_pool_pages_dirty        | 41880                                            |
   | Innodb_buffer_pool_bytes_dirty        | 686161920                                        |
   | Innodb_buffer_pool_pages_flushed      | 83319320                                         |
   | Innodb_buffer_pool_pages_free         | 8191                                             |
   | Innodb_buffer_pool_pages_misc         | 9831                                             |
   | Innodb_buffer_pool_pages_total        | 655360                                           |
   | Innodb_buffer_pool_read_ahead_rnd     | 0                                                |
   | Innodb_buffer_pool_read_ahead         | 0                                                |
   | Innodb_buffer_pool_read_ahead_evicted | 0                                                |
   | Innodb_buffer_pool_read_requests      | 990262384355                                     |
   | Innodb_buffer_pool_reads              | 28819853                                         |
   | Innodb_buffer_pool_wait_free          | 3810                                             |
   | Innodb_buffer_pool_write_requests     | 14201720865                                      |
   | Innodb_data_fsyncs                    | 667145550                                        |
   | Innodb_data_pending_fsyncs            | 0                                                |
   | Innodb_data_pending_reads             | 0                                                |
   | Innodb_data_pending_writes            | 0                                                |
   | Innodb_data_read                      | 472202646016                                     |
   | Innodb_data_reads                     | 29303659                                         |
   | Innodb_data_writes                    | 716111575                                        |
   | Innodb_data_written                   | 5004313035776                                    |
   | Innodb_dblwr_pages_written            | 78031611                                         |
   | Innodb_dblwr_writes                   | 11080229                                         |
   | Innodb_log_waits                      | 0                                                |
   | Innodb_log_write_requests             | 2970752813                                       |
   | Innodb_log_writes                     | 620367623                                        |
   | Innodb_os_log_fsyncs                  | 620811810                                        |
   | Innodb_os_log_pending_fsyncs          | 0                                                |
   | Innodb_os_log_pending_writes          | 0                                                |
   | Innodb_os_log_written                 | 2352685154816                                    |
   | Innodb_page_size                      | 16384                                            |
   | Innodb_pages_created                  | 14678150                                         |
   | Innodb_pages_read                     | 28823721                                         |
   | Innodb_pages_written                  | 83809344                                         |
   | Innodb_row_lock_current_waits         | 186                                              |
   | Innodb_row_lock_time                  | 8633275156                                       |
   | Innodb_row_lock_time_avg              | 83                                               |
   | Innodb_row_lock_time_max              | 6985                                             |
   | Innodb_row_lock_waits                 | 103575326                                        |
   | Innodb_rows_deleted                   | 96084957                                         |
   | Innodb_rows_inserted                  | 931399665                                        |
   | Innodb_rows_read                      | 2496719485217                                    |
   | Innodb_rows_updated                   | 686287179                                        |
   | Innodb_num_open_files                 | 65535                                            |
   | Innodb_truncated_status_writes        | 0                                                |
   | Innodb_available_undo_logs            | 128                                              |
   | Innodb_column_compressed              | 0                                                |
   | Innodb_column_decompressed            | 0                                                |
   | Innodb_column_cmps_before             | 0                                                |
   | Innodb_column_cmps_after              | 0                                                |
   | Innodb_column_cmps_percent            | 0                                                |
   +---------------------------------------+--------------------------------------------------+
   56 rows in set (0.01 sec)
   ```
   
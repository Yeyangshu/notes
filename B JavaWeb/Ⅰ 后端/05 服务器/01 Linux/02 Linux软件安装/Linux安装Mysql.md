https://www.cnblogs.com/shidian/p/11589626.html

1. mysql下载地址：https://downloads.mysql.com/archives/community/

2. 上传压缩包至服务器并解压文件

   ```
   tar -zxvf mysql-5.7.30-linux-glibc2.12-x86_64.tar_213.gz
   ```

3. 移动解压后的文件并重命名

   ```
   mv mysql-5.7.30-linux-glibc2.12-x86_64 /usr/local/mysql
   ```

4. 创建MySQL用户组合用户并修改权限

   ```
   groupadd mysql
   useradd -r -g mysql mysql
   ```

5. 创建数据目录并赋予权限

   ```
   mkdir -p  /usr/local/mysql/data              #创建目录
   chown mysql:mysql -R /usr/local/mysql/data   #赋予权限
   ```

6. 新建配置文件

   ```
   vi /etc/my.cnf
   
   [mysqld]
   bind-address=0.0.0.0
   port=3306
   user=mysql
   basedir=/usr/local/mysql
   datadir=/usr/local/mysql/data
   socket=/tmp/mysql.sock
   log-error=/usr/local/mysql/mysql.err
   pid-file=/usr/local/mysql/mysql.pid
   #character config
   character_set_server=utf8mb4
   symbolic-links=0
   explicit_defaults_for_timestamp=true
   ```

7. 初始化数据库

   ```
   # 进入bin目录
   cd /usr/local/mysql/bin/
   
   # 初始化，目录要和配置文件保持一致
   ./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data --user=mysql --initialize
   
   # 查看密码
   cat mysql.err | grep password
   ```

   ![image-20200808082108508](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20200808082108508.png)

8. 启动mysql

   ```
   # 将mysql.server放置到/etc/init.d/mysql中
   cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
   
   # 启动
   service mysql start
   
   # 开机自启
   chkconfig mysql on
   ```

9. 修改密码

   ```
   ./mysql -u root -p   #bin目录下
   
   # 执行下面三步操作，然后重新登陆
   SET PASSWORD = PASSWORD('123456');
   ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
   FLUSH PRIVILEGES;
   ```

10. 远程登陆

    ```
    use mysql                                            #访问mysql库
    update user set host = '%' where user = 'root';      #使root能再任何host访问
    FLUSH PRIVILEGES;                                    #刷新
    ```

    如果还不行，重启一下mysql服务

11. 如果不希望每次进到bin目录下使用mysql命令，则执行一下命令。

    ```
    ln -s  /usr/local/mysql/bin/mysql    /usr/bin
    ```

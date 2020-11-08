# RocketMQ

## 1 安装与启动

### 1.1 RocketMQ安装

1. 下载地址：http://rocketmq.apache.org/dowloading/releases/，下载并解压文件

   rocketmq-all-4.7.1-bin-release.zip

2. 设置环境变量

   ```
   ROCKETMQ_HOME=文件解压根目录
   ```

3. cmd进入目录，输入命令

   ```powershell
   start mqnamesrv.cmd
   ```

### 1.2 rocketmq-externals

1. git clone https://github.com/apache/rocketmq-externals，下载文件

2. `rocketmq-externals\rocketmq-console`目录下编译打包

   ```
   mvn clean package -Dmaven.test.skip=true
   ```

3. 进入target文件夹运行jar包，执行

   ```
   java -jar rocketmq-console-ng-1.0.1.jar
   ```

4. 访问http://localhost:1234，即可查看控制台信息
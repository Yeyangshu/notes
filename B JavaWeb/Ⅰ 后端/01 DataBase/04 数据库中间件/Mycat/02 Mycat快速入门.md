# Mycat 快速入门

## 1 10分钟入门

1.1 环境准备

1.2 环境安装与配置

## 2  快速镜像方式体验MyCAT

## 3 服务安装与配置

### 3.2 Windows

Windows 下可以下载 Mycat-server-xxxxx-win.tar.gz 解压在某个目录下

目录解释如下：

- bin 程序目录，存放了window版本和linux版本，除了提供封装成服务的版本之外，也提供了nowrap的shell脚本命令，方便大家选择和修改，进入到bin目录：

- Windows下运行：运行: mycat.bat在控制台启动程序，也可以装载成服务，若此程序运行有问题，也可以运行startup_nowrap.bat，确保java命令可以在命令执行。

- Windows下将MyCAT做成系统服务：MyCAT提供warp方式的命令，可以将MyCAT安装成系统服务并可启动和停止。
  - 进入bin目录下， 输入 ./mycat start 启动mycat服务。

- conf目录下存放配置文件，server.xml是Mycat服务器参数调整和用户授权的配置文件，schema.xml是逻辑库定义和表以及分片定义的配置文件，rule.xml是分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下，配置文件修改，需要重启Mycat或者通过9066端口reload。

- lib目录下主要存放mycat依赖的一些jar文件。

- 日志存放在logs/mycat.log中，每天一个文件，日志的配置是在conf/log4j.xml中，根据自己的需要，可以调整输出级别为debug，debug级别下，会输出更多的信息，方便排查问题。
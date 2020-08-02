

上传文件并解压

设置环境变量

```
vi /etc/profile

//文件末尾添加环境变量
#set java environment
export JAVA_HOME=/opt/soft/jdk1.8.0_181

export PATH=$PATH:$JAVA_HOME/bin
```

使配置生效

```
source /etc/profile
```

测试

```
java -version
```


# Linux下安装

**下载**

**解压**到

在`init.d`下建立软连接

```
ln -s /usr/local/activemq/bin/activemq ./
```

**设置开启启动**

`chkconfig activemq on`

服务管理

```
service activemq start
service activemq status
service activemq stop
```


# Active MQ的安全机制

## 1 web控制台安全

```
# username: password [,rolename ...]
admin: admin, admin
user: user, user
yiming: 123, user
```

用户名：密码，角色

注意：配置需重启ActiveMQ才会生效。

## 2 消息安全机制

修改 activemq.xml

在`</broker> `节点中添加

```xml
<plugins>
    <simpleAuthenticationPlugin>
        <users>
            <authenticationUser username="admin" password="admin" groups="admins,publishers,consumers"/>
            <authenticationUser username="publisher" password="publisher"  groups="publishers,consumers"/>
            <authenticationUser username="consumer" password="consumer" groups="consumers"/>
            <authenticationUser username="guest" password="guest"  groups="guests"/>
        </users>
    </simpleAuthenticationPlugin>
</plugins>

```

![image-20201107203430421](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201107203430421.png)

重启

![image-20201107203916072](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201107203916072.png)

此时如果还使用上面的配置会报以下错误

![image-20201107203846320](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201107203846320.png)
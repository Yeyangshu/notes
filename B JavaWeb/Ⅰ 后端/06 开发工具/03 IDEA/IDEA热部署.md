# IDEA热部署

pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>true</scope>
</dependency>
```

pom插件

```xml
<plugin>
    <!--热部署配置-->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!--fork:如果没有该项配置,整个devtools不会起作用-->
        <fork>true</fork>
    </configuration>
</plugin>
```

勾选Compiler下的Build Project automatically

```
File | Settings | Build, Execution, Deployment | Compiler
```

勾选 ctrl + shift + alt + / ，Registry...，complier.automake.allow.when.app.running
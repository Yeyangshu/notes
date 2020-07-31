# Mybatis配置文件详解

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--引入外部配置文件，类似于Spring中的property-placeholder
    resource:从类路径引入
    url:从磁盘路径或者网络路径引入
    -->
    <properties resource="db.properties"></properties>
    <!--用来控制mybatis运行时的行为，是mybatis中的重要配置-->
    <settings>
        <!--设置列名映射的时候是否是驼峰标识-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <!--typeAliases表示为我们引用的实体类起别名，默认情况下我们需要写类的完全限定名
    如果在此处做了配置，那么可以直接写类的名称,在type中配置上类的完全限定名，在使用的时候可以忽略大小写
    还可以通过alias属性来表示类的别名
    -->
    <typeAliases>
<!--        <typeAlias type="com.mashibing.bean.Emp" alias="Emp"></typeAlias>-->
        <!--如果需要引用多个类，那么给每一个类起别名肯定会很麻烦，因此可以指定对应的包名，那么默认用的是类名-->
        <package name="com.mashibing.bean"/>
    </typeAliases>
    <!--
    在实际的开发过程中，我们可能分为开发环境，生产环境，测试环境等等，每个环境的配置可以是不一样的
    environment就用来表示不同环境的细节配置，每一个环境中都需要一个事务管理器以及数据源的配置
    我们在后续的项目开发中几乎都是使用spring中配置的数据源和事务管理器来配置，此处不需要研究
    -->
    <!--default:用来选择需要的环境-->
    <environments default="development">
        <!--id:表示不同环境的名称-->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--配置数据库连接-->
            <dataSource type="POOLED">
                <!--使用${}来引入外部变量-->
                <property name="driver" value="${driverClassname}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--
    在不同的数据库中，可能sql语句的写法是不一样的，为了增强移植性，可以提供不同数据库的操作实现
    在编写不同的sql语句的时候，可以指定databaseId属性来标识当前sql语句可以运行在哪个数据库中
    -->
    <databaseIdProvider type="DB_VENDOR">
        <property name="MySQL" value="mysql"/>
        <property name="SQL Server" value="sqlserver"/>
        <property name="Oracle" value="orcl"/>
    </databaseIdProvider>
    
    <!--将sql的映射文件适用mappers进行映射-->
    <mappers>
        <!--
        指定具体的不同的配置文件
        class:直接引入接口的全类名，可以将xml文件放在dao的同级目录下，并且设置相同的文件名称，同时可以使用注解的方式来进行相关的配置
        url:可以从磁盘或者网络路径查找sql映射文件
        resource:在类路径下寻找sql映射文件
        -->
<!--        <mapper resource="EmpDao.xml"/>
        <mapper resource="UserDao.xml"/>
        <mapper class="com.mashibing.dao.EmpDaoAnnotation"></mapper>-->
        <!--
        当包含多个配置文件或者配置类的时候，可以使用批量注册的功能，也就是引入对应的包，而不是具体的配置文件或者类
        但是需要注意的是，
        1、如果使用的配置文件的形式，必须要将配置文件跟dao类放在一起，这样才能找到对应的配置文件.
            如果是maven的项目的话，还需要添加以下配置，原因是maven在编译的文件的时候只会编译java文件
                <build>
                    <resources>
                        <resource>
                            <directory>src/main/java</directory>
                        <includes>
                            <include>**/*.xml</include>
                        </includes>
                    </resource>
                    </resources>
                </build>

        2、将配置文件在resources资源路径下创建跟dao相同的包名
        -->
        <package name="com.mashibing.dao"/>
    </mappers>
</configuration>
```




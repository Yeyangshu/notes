## 遇到的问题

parameterMap="moduleMap" 更改为parameterType="com.demo.sys.entity.Module" 解决了Mybatis中Parameter Maps collection does not contain value for xxx 的问题了。

 

查看Mybatis官方资料;

 SQL映射的XML文件：parameterMap 已经废弃了，现在使用parameterType来处理。

resultMap使用正常
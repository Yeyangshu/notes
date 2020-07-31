# Mybatis映射文件

## 1 insert, update 和 delete属性

| 属性                 | 描述                                                         |
| :------------------- | ------------------------------------------------------------ |
| **id**               | 用来标识跟dao接口中匹配的方法，必须与方法名字一一对应        |
| **flashCache**       | 用来标识当前sql语句的结果是否进入二级缓存                    |
| statementType        | 可选 STATEMENT，PREPARED 或 CALLABLE。<br />STATEMENT：最基本的jdbc操作，用来表示一个sql语句，不能防止sql注入<br />PREPARED：采用预编译的方式，能够防止sql注入，设置参数的时候需要该对象来进行设置 |
| **useGeneratedKeys** | 完成插入的时候，可以将自增生成的主键值返回到具体的对象       |
| **keyProperty**      | 指定要返回的主键要赋值到那个属性中                           |
|                      |                                                              |
|                      |                                                              |

使用useGeneratedKeys和keyProperty要使用user.getId()进行取值

## 2 select

在mybatis中，会包含非常多的查询操作，因此查询的操作非常麻烦

| 属性           | 描述 |
| -------------- | ---- |
| **id**         |      |
| **resultType** |      |
| **resultMap**  |      |

### 2.1 select的参数传递

当查询语句中包含多个参数的时候，我们应该如何获取需要的参数

#### 2.1.1 单个参数

- 基本数据类型：那么可以使用#{}随便获取

  dao

  ```java
  Emp emp = mapper.findEmpByEmpno(7369);
  ```

  sql

  ```sql
  select * from emp where empno = #{1}
  ```

- 引用数据类型：适用#{}获取值的时候必须要使用对象的属性名

  dao

  ```java
  public List<Emp> selectEmpByEmpnoAndSql(Emp emp);
  ```

  sql

  ```xml
  <select id="selectEmpByEmpnoAndSql" resultType="Emp">
      select * from emp where empno = #{empno} and sal >= #{sal}
  </select>
  ```

#### 2.1.2 多个参数

我们在获取参数的时候，就不能简单的通过#{}来获取了，只能通过arg0，arg1，param1，param2...这样的方式来货物参数的值，原因在于mybatis在传入多个参数的时候会将这些参数的结果封装到map结构中，在map中key的值就是{arg0, arg1...}，这种方式非常不友好，没有办法根据属性名来获取具体的参数值，如果想使用参数的话，可以进行如下的设置

```java
public List<Emp> selectEmpByEmpnoAndSal2(@Param("empno") Integer empno, @Param("sal") Double sal);
```

这样的方式其实是根据@Param来进行参数的获取

#### 2.1.3 自定义map结构

dao

```java
public List<Emp> selectEmpByEmpnoAndSql3(Map<String, Object> map);
```

sql

```xml
<select id="selectEmpByEmpnoAndSql3" resultType="Emp">
    select * from emp where empno = #{empno} and ename = #{ename}
</select>
```

test

```java
Map<String, Object> map = new HashMap<>();
map.put("empno", 7369);
map.put("ename", "SMITH");
List<Emp> empList2 = mapper.selectEmpByEmpnoAndSql3(map);
```

### 2.2 参数的取值方式

每次在向sql语句中设置结果值的时候，有两种取值的方式，可以使用#{}、${}的方式，哪一种方式比较好呢？

通过sql语句可以得出结论：

- #{}：select * from emp where empno = ?
- ${}：select * from emp where empno = 7369

#{}方式是使用了参数预编译的方式，可以防止sql注入的问题

${}方式是使用了直接拼接sql语句，得到对应的sql语句，会有sql注入的问题

因此，推荐大家使用#{}的方式

但是${}也是有自己的适用场景的：当需要传入动态的表名，列名的时候就需要使用${}，就是最直接的拼接字符串的行为

### 2.3 处理集合返回结果

#### 2.3.1 返回值是List集合

当返回值是List集合时，返回值类型写集合中的类型

dao

```java
List<Emp> selectAllEmp();
```

sql

```xml
<select id="selectAllEmp" resultType="com.yeyangshu.bean.Emp">
    select * from emp
</select>
```

结果

```
[Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, deptno=20}, Emp{empno=7499, ename='ALLEN', job='SALESMAN', mgr=7698, hiredate=Fri Feb 20 08:00:00 CST 1981, sal=1600.0, common=null, deptno=30}, Emp{empno=7521, ename='WARD', job='SALESMAN', mgr=7698, hiredate=Sun Feb 22 08:00:00 CST 1981, sal=1250.0, common=null, deptno=30}, Emp{empno=7566, ename='JONES', job='MANAGER', mgr=7839, hiredate=Thu Apr 02 08:00:00 CST 1981, sal=2975.0, common=null, deptno=20}, Emp{empno=7654, ename='MARTIN', job='SALESMAN', mgr=7698, hiredate=Mon Sep 28 08:00:00 CST 1981, sal=1250.0, common=null, deptno=30}, Emp{empno=7698, ename='BLAKE', job='MANAGER', mgr=7839, hiredate=Fri May 01 08:00:00 CST 1981, sal=2850.0, common=null, deptno=30}, Emp{empno=7782, ename='CLARK', job='MANAGER', mgr=7839, hiredate=Tue Jun 09 08:00:00 CST 1981, sal=2450.0, common=null, deptno=10}, Emp{empno=7788, ename='SCOTT', job='ANALYST', mgr=7566, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=3000.0, common=null, deptno=20}, Emp{empno=7839, ename='KING', job='PRESIDENT', mgr=null, hiredate=Sat Nov 07 08:00:00 CST 1981, sal=5000.0, common=null, deptno=10}, Emp{empno=7844, ename='TURNER', job='SALESMAN', mgr=7698, hiredate=Tue Sep 08 08:00:00 CST 1981, sal=1500.0, common=null, deptno=30}, Emp{empno=7876, ename='ADAMS', job='CLERK', mgr=7788, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=1100.0, common=null, deptno=20}, Emp{empno=7900, ename='JAMES', job='CLERK', mgr=7698, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=950.0, common=null, deptno=30}, Emp{empno=7902, ename='FORD', job='ANALYST', mgr=7566, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=3000.0, common=null, deptno=20}, Emp{empno=7934, ename='MILLER', job='CLERK', mgr=7782, hiredate=Sat Jan 23 08:00:00 CST 1982, sal=1300.0, common=null, deptno=10}]
```

#### 2.3.2 返回值的类型为map

当mybatis查询完成之后会把列的名称作为key，列的值作为value，转换到map中，例ename=SMITH

dao

```java
Map<String, Object> selectEmpByEmpReturnMap(Integer empno);
```

sql

```xml
<select id="selectEmpByEmpReturnMap" resultType="map">
    select * from emp where empno = #{empno}
</select>
```

结果

```
{ename=SMITH, mgr=7902, empno=7369, job=CLERK, hiredate=1980-12-17, deptno=20, sal=800.0}
```

#### 2.3.3 返回的结果是一个集合对象

当返回的结果是一个集合对象，返回值的类型一定要写集合具体value的类型，同时在dao的方法上要添加@MapKey的注解，来设置key值

dao

```java
@MapKey("empno")
Map<Integer,Emp> getAllEmpReturnMap();
```

sql

```xml
<select id="getAllEmpReturnMap" resultType="Emp">
    select * from emp
</select>
```

结果

```json
{7521=Emp{empno=7521, ename='WARD', job='SALESMAN', mgr=7698, hiredate=Sun Feb 22 08:00:00 CST 1981, sal=1250.0, common=null, deptno=30}, 7844=Emp{empno=7844, ename='TURNER', job='SALESMAN', mgr=7698, hiredate=Tue Sep 08 08:00:00 CST 1981, sal=1500.0, common=null, deptno=30}, 7876=Emp{empno=7876, ename='ADAMS', job='CLERK', mgr=7788, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=1100.0, common=null, deptno=20}, 7654=Emp{empno=7654, ename='MARTIN', job='SALESMAN', mgr=7698, hiredate=Mon Sep 28 08:00:00 CST 1981, sal=1250.0, common=null, deptno=30}, 7782=Emp{empno=7782, ename='CLARK', job='MANAGER', mgr=7839, hiredate=Tue Jun 09 08:00:00 CST 1981, sal=2450.0, common=null, deptno=10}, 7369=Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, deptno=20}, 7499=Emp{empno=7499, ename='ALLEN', job='SALESMAN', mgr=7698, hiredate=Fri Feb 20 08:00:00 CST 1981, sal=1600.0, common=null, deptno=30}, 7788=Emp{empno=7788, ename='SCOTT', job='ANALYST', mgr=7566, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=3000.0, common=null, deptno=20}, 7566=Emp{empno=7566, ename='JONES', job='MANAGER', mgr=7839, hiredate=Thu Apr 02 08:00:00 CST 1981, sal=2975.0, common=null, deptno=20}, 7698=Emp{empno=7698, ename='BLAKE', job='MANAGER', mgr=7839, hiredate=Fri May 01 08:00:00 CST 1981, sal=2850.0, common=null, deptno=30}, 7900=Emp{empno=7900, ename='JAMES', job='CLERK', mgr=7698, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=950.0, common=null, deptno=30}, 7902=Emp{empno=7902, ename='FORD', job='ANALYST', mgr=7566, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=3000.0, common=null, deptno=20}, 7934=Emp{empno=7934, ename='MILLER', job='CLERK', mgr=7782, hiredate=Sat Jan 23 08:00:00 CST 1982, sal=1300.0, common=null, deptno=10}, 7839=Emp{empno=7839, ename='KING', job='PRESIDENT', mgr=null, hiredate=Sat Nov 07 08:00:00 CST 1981, sal=5000.0, common=null, deptno=10}}
```

### 2.4 自定义结果集—resultMap

Dog类

```java
public class Dog {
    private Integer id;
    private String name;
    private Integer age;
    private String gender;
}
```

数据库表

```
id
dname
dage
dgender
```

在使用mybatis进行查询的时候，mybatis默认会帮我们进行结果的封装，但是要求列名跟属性名称一一对应上

在实际的使用过程中，我们会发现有时候数据库中的列名跟我们类中的属性名并不是一一对应的，此时就需要起别名

起别名有两种实现方式：

1. 在编写sql语句的时候添加别名
2. 自定义封装结果集



数据库与Javabean名称不符案例

dao

```java
public Dog selectDogById(Integer id);
```

sql

```xml
<select id="selectDogById" resultType="com.yeyangshu.mybatis.bean.Dog">
    select * from dog where id = #{id}
</select>
```

结果部分为空

```
Dog{id=1, name='null', age=null, gender='null'}
```

#### 2.4.1 sql别名

缺点：麻烦，不可重用

dao

```java
public Dog selectDogById(Integer id);
```

sql

```xml
<select id="selectDogById" resultType="com.yeyangshu.mybatis.bean.Dog">
    select id id, dname name, dage age, dgender gender from dog where id = #{id}
</select>
```

结果

```json
Dog{id=1, name='大黄', age=1, gender='雄'}
```

#### 2.4.2 自定义结果集

resultMap

```xml
<!--
    自定义结果集，将每一个列的数据跟javaBean的对象属性对应起来
    type:表示为哪一个javaBean对象进行对应
    id:唯一标识，方便其他属性标签进行引用
-->
<resultMap id="myDog" type="com.yeyangshu.mybatis.bean.Dog">
    <!--
        指定主键列的对应规则：
        column：表示表中的主键列
        property:指定javaBean的属性
		javaType:
		jdbcType:
    -->
    <id column="id" property="id"></id>
    <!--设置其他列的对应关系-->
    <result column="dname" property="name"></result>
    <result column="dage" property="age"></result>
    <result column="dgender" property="gender"></result>
</resultMap>
```

sql

```xml
<select id="selectDogById" resultMap="myDog">
    select * from dog where id = #{id}
</select>
```

结果

```json
Dog{id=1, name='大黄', age=1, gender='雄'}
```

### 2.5 联合查询

在做查询的时候，有时候需要关联其他**对象**，因此需要使用关联查询

Emp.java

```java
public class Emp {

    private Integer empno;
    private String ename;
    private String job;
    private Integer mgr;
    private Date hiredate;
    private Double sal;
    private Double common;
    private Dept dept;
}
```

Dept.java

```java
public class Dept {
    private Integer deptno;
    private String dname;
    private String loc;
}
```

sql

```xml
<select id="selectEmpByEmpno" resultMap="myEmp">
    select * from emp left join dept on emp.deptno = dept.deptno where empno = #{empno};
</select>
```

#### 2.5.1 对象.属性

将每一种属性值都映射成为对象中的数据，如果有实体类对象，就写成对象.属性的方式

resultMap

```xml
<resultMap id="myEmp" type="com.yeyangshu.mybatis.bean.Emp">
    <id column="empno" property="empno"></id>
    <result column="ename" property="ename"></result>
    <result column="job" property="job"></result>
    <result column="mgr" property="mgr"></result>
    <result column="hiredate" property="hiredate"></result>
    <result column="sal" property="sal"></result>
    <result column="comm" property="common"></result>
    <result column="deptno" property="dept.deptno"></result>
    <result column="dname" property="dept.dname"></result>
    <result column="loc" property="dept.loc"></result>
</resultMap>
```

结果

```
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=Dept{deptno=20, dname='RESEARCH', loc='DALLAS'}}
```

#### 2.5.2 association

resultMap

```xml
<resultMap id="myEmp" type="com.yeyangshu.mybatis.bean.Emp">
    <id column="empno" property="empno"></id>
    <result column="ename" property="ename"></result>
    <result column="job" property="job"></result>
    <result column="mgr" property="mgr"></result>
    <result column="hiredate" property="hiredate"></result>
    <result column="sal" property="sal"></result>
    <result column="comm" property="common"></result>
    <association property="dept" javaType="com.yeyangshu.mybatis.bean.Dept">
        <id column="deptno" property="deptno"/>
        <result column="dname" property="dname"/>
        <result column="loc" property="loc"/>
    </association>
</resultMap>
```

结果

```json
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=Dept{deptno=20, dname='RESEARCH', loc='DALLAS'}}
```

#### 2.5.3 关联的嵌套 Select 查询

与join对比

```mysql
select * from dept left join emp on dept.deptno = emp.deptno where dept.deptno = #{deptno}
```



| **属性**  | **描述**                                                     |
| --------- | ------------------------------------------------------------ |
| column    |                                                              |
| select    | 用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句。 具体请参考下面的例子。注意：在使用复合主键的时候，你可以使用 column="{prop1=col1,prop2=col2}" 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 prop1 和 prop2 作为参数对象，被设置为对应嵌套 Select 语句的参数。 |
| fetchType |                                                              |

DeptDao

```java
public Dept getDeptAndEmpsBySimple(Integer deptno);
```

DeptDao.xml

```xml
<select id="getDeptAndEmpsBySimple" resultType="com.yeyangshu.mybatis.bean.Dept">
    select * from dept where deptno = #{deptno}
</select>
```

EmpDao

```java
Emp selectEmpAndDeptBySimple(Integer empno);
```

EmpDao.xml

```xml
<resultMap id="simpleEmpAndDept" type="com.yeyangshu.mybatis.bean.Emp">
    <id column="empno" property="empno"></id>
    <result column="ename" property="ename"></result>
    <result column="job" property="job"></result>
    <result column="mgr" property="mgr"></result>
    <result column="hiredate" property="hiredate"></result>
    <result column="sal" property="sal"></result>
    <result column="comm" property="common"></result>
    <association property="dept" select="com.yeyangshu.mybatis.dao.DeptDao.getDeptAndEmpsBySimple" column="deptno">
    </association>
</resultMap>
```

test

```java
Emp emp = empDao.selectEmpAndDeptBySimple(7369);
```

结果

```json
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=Dept{deptno=20, dname='RESEARCH', loc='DALLAS', emps=null}}xxxxxxxxxx [root]Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=Dept{deptno=20, dname='RESEARCH', loc='DALLAS', emps=null}}json
```



### 2.6 获取集合元素—collection

#### 2.6.1 collection

Emp.java

```java
public class Emp {

    private Integer empno;
    private String ename;
    private String job;
    private Integer mgr;
    private Date hiredate;
    private Double sal;
    private Double common;
    private Dept dept;
}
```

Dept.java

```java
public class Dept {
    private Integer deptno;
    private String dname;
    private String loc;
    
    private List<Emp> emps;
}
```

DeptDao.java

```java
public Dept getDeptAndEmps(Integer deptno);
```

DeptDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yeyangshu.mybatis.dao.DeptDao">
    <!--定义查询集合元素-->
    <select id="getDeptAndEmps" resultMap="deptEmp">
        select * from dept left join emp on dept.deptno = emp.deptno where dept.deptno = #{deptno}
    </select>
    <resultMap id="deptEmp" type="com.yeyangshu.mybatis.bean.Dept">
        <id property="deptno" column="deptno"></id>
        <result property="dname" column="dname"></result>
        <result property="loc" column="loc"></result>
        <!--封装集合类的元素
            property：指定集合的属性
            ofType：指定集合中的元素类型
        -->
        <collection property="emps" ofType="com.yeyangshu.mybatis.bean.Emp">
            <id property="empno" column="empno"></id>
            <result column="ename" property="ename"></result>
            <result column="job" property="job"></result>
            <result column="mgr" property="mgr"></result>
            <result column="hiredate" property="hiredate"></result>
            <result column="sal" property="sal"></result>
            <result column="comm" property="common"></result>
        </collection>
    </resultMap>
</mapper>
```

结果

```json
Dept{deptno=20, dname='RESEARCH', loc='DALLAS', emps=[Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}, Emp{empno=7566, ename='JONES', job='MANAGER', mgr=7839, hiredate=Thu Apr 02 08:00:00 CST 1981, sal=2975.0, common=null, dept=null}, Emp{empno=7788, ename='SCOTT', job='ANALYST', mgr=7566, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=3000.0, common=null, dept=null}, Emp{empno=7876, ename='ADAMS', job='CLERK', mgr=7788, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=1100.0, common=null, dept=null}, Emp{empno=7902, ename='FORD', job='ANALYST', mgr=7566, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=3000.0, common=null, dept=null}]}
```

#### 2.6.2 集合的嵌套 Select 查询



## 3 缓存

Mybatis的缓存机制：

如果没有缓存，每次查询的时候都需要从数据库中加载数据，会造成io的性能问题，所以，在很多情况下，连续执行两条相同的sql语句，可以直接从缓存中获取，如果获取不到，那么再去查数据库，这意味着查询完成的结果会放到缓存中。

Mybatis的缓存分类：

1. 一级缓存：表示sqlSession级别的缓存，每次查询的时候会开启一个会话，此会话相当于一次连接，关闭之后自动失效
2. 二级缓存：全局范围内的缓存，sqlSession关闭之后才会生效
3. 第三方缓存：继承第三方的组件，来充当缓存的作用

Mybatis 内置了一个强大的事务性查询缓存机制，它可以非常方便地配置和定制。 为了使它更加强大而且易于配置，我们对 MyBatis 3 中的缓存实现进行了许多改进。

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

```xml
<cache/>
```

### 3.1 一级缓存

表示数据存储在sqlSession中，关闭之后自动失效，默认情况下是开启的

在同一个会话之内，如果执行了多个相同的sql语句，那么除了第一个之外，所有的数据都是从缓存中进行查询的

sql

```xml
<select id="selectEmp" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp where empno = #{empno}
</select>
```

test

```java
Emp emp2 = empDao.selectEmp(7369);
System.out.println(emp2);
Emp emp3 = empDao.selectEmp(7369);
System.out.println(emp3);
sqlSession.close();
```

result

```
09:50:14.113 [main] DEBUG com.yeyangshu.mybatis.dao.EmpDao.selectEmp - ==>  Preparing: select * from emp where empno = ? 
09:50:14.143 [main] DEBUG com.yeyangshu.mybatis.dao.EmpDao.selectEmp - ==> Parameters: 7369(Integer)
09:50:14.170 [main] DEBUG com.yeyangshu.mybatis.dao.EmpDao.selectEmp - <==      Total: 1
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}
```



**在一些情况，一级缓存可能会失效**

- 在同一个方法中可能会开启多个会话，注意，会话和方法没有关系，不是一个方法只能有一个会话，所以一定记住，缓存的数据是保存在Session里面的 

  ```java
  SqlSession sqlSession = sqlSession7         Factory.openSession(true);
  Emp emp = empDao.selectEmp(7369);
  System.out.println(emp);
  SqlSession sqlSession2 = sqlSessionFactory.openSession(true);
  Emp emp2 = empDao.selectEmp(7369);
  System.out.println(emp2);
  此时session2不会使用session的结果
  ```

- 当传递对象的时候，如果对象中的属性不同，也不会走缓存

  ```java
  empDao.selectEmpByCondition(emp)
  ```

- 在多次查询过程中，如果修改了数据，缓存会失效

- 如果在一个会话中，手动清空了缓存，那么缓存也会失效

  ```java
  sqlSession.clearCache();
  ```

- 
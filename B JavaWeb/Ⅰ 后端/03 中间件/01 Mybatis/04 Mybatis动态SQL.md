# Mybatis动态SQL

动态SQL元素

- if
- choose(when, otherwise)
- trim(where, set)
- foreach



## 1 if

根据条件包含 where 子句的一部分

dao

```java
public Emp selectEmpByCondition(Emp emp);
```

sql

```xml
<select id="selectEmpByCondition" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp where
    <if test="empno != null">
        empno = #{empno} and
    </if>
    <if test="ename != null">
        ename = #{ename}
    </if>
</select>
```

test

```java
Emp emp = new Emp();
emp.setEmpno(7369);
emp.setEname("SMITH");
System.out.println(empDao.selectEmpByCondition(emp));
```

result

```json
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}
```

动态sql如下

```sql
select * from emp where empno = ? and ename = ? 
```

问题：如果去掉emp.setEname("SMITH")，会发生异常，动态sql如下

```sql
select * from emp where empno = ? and
```

使用where元素解决

## 2 where

where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。

sql

```xml
<select id="selectEmpByCondition" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp
    <where>
        <if test="empno != null">
            empno = #{empno}
        </if>
        <if test="ename != null">
            and ename = #{ename}
        </if>
    </where>
</select>
```

test

```java
emp.setEmpno(7369);
System.out.println(empDao.selectEmpByCondition(emp));
```

动态sql

```sql
select * from emp WHERE empno = ?
```

结果

```json
Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}
```

## 3 choose、when、otherwise

sql

```xml
<select id="selectEmpByCondition" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp
    <where>
        <choose>
            <when test="empno != null">
                empno = #{empno}
            </when>
            <when test="ename != null">
                and ename = #{ename}
            </when>
            <otherwise>
                and 1 = 1
            </otherwise>
        </choose>
    </where>
</select>
```

test

```java
Emp emp = new Emp();
emp.setEmpno(7369);
emp.setEname("SMITH");
emp.setSal(1000.0);
System.out.println(empDao.selectEmpByCondition(emp));
```

动态SQL

```sql
select * from emp WHERE empno = ? 
```

对比if

```sql
select * from emp WHERE empno = ? and ename = ? and sal > ? 
```

## 4 trim、set（where）

### 4.1 trim

- trim：截取字符串，可以自定义where格式
- prefix：为sql整体添加一个前缀
- prefixOverrides：去除整体sql语句前面多余的字符串
- suffixOverrides：去除整体sql语句后面多余的字符串

prefixOverrides 属性会忽略通过管道符分隔的文本序列（注意此例中的空格是必要的）。移除所有prefixOverrides 属性中指定的内容，并且插入 prefix 属性中指定的内容。

sql

**注意：**and empno = #{empno}前的and，and sal > #{sal} or后的or

```xml
<select id="selectEmpByCondition" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp
    <trim prefix="where" prefixOverrides="and" suffixOverrides="and | or">
        <if test="empno != null">
            and empno = #{empno}
        </if>
        <if test="ename != null">
            and ename = #{ename}
        </if>
        <if test="sal != null">
            and sal > #{sal} or
        </if>
    </trim>
</select>
```

test

```java
Emp emp = new Emp();
emp.setEmpno(7369);
emp.setEname("SMITH");
emp.setSal(1000.0);
System.out.println(empDao.selectEmpByCondition(emp));
```

动态sql

```sql
select * from emp where empno = ? and ename = ? and sal > ? 
```

## 5 foreach

参数

- foreach：遍历集合中的标签
- separator：分隔符
- open：以什么开始
- close：以什么结束
- item：遍历过程中的每一个元素值
- index：索引

dao

```java
public List<Emp> selectEmpsByDeptnos(@Param("deptnos")List<Integer> deptnos);
```

sql

**注意：item="deptno"与#{deptno}里面的deptno要一致**

```xml
<select id="selectEmpsByDeptnos" resultType="com.yeyangshu.mybatis.bean.Emp">
    select * from emp where deptno in
    <foreach collection="deptnos" separator="," open="(" close=")" item="deptno" index="idx">
        #{deptno}
    </foreach>
</select>
```

结果

```json
[Emp{empno=7369, ename='SMITH', job='CLERK', mgr=7902, hiredate=Wed Dec 17 08:00:00 CST 1980, sal=800.0, common=null, dept=null}, Emp{empno=7566, ename='JONES', job='MANAGER', mgr=7839, hiredate=Thu Apr 02 08:00:00 CST 1981, sal=2975.0, common=null, dept=null}, Emp{empno=7782, ename='CLARK', job='MANAGER', mgr=7839, hiredate=Tue Jun 09 08:00:00 CST 1981, sal=2450.0, common=null, dept=null}, Emp{empno=7788, ename='SCOTT', job='ANALYST', mgr=7566, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=3000.0, common=null, dept=null}, Emp{empno=7839, ename='KING', job='PRESIDENT', mgr=null, hiredate=Sat Nov 07 08:00:00 CST 1981, sal=5000.0, common=null, dept=null}, Emp{empno=7876, ename='ADAMS', job='CLERK', mgr=7788, hiredate=Mon Jul 13 09:00:00 CDT 1987, sal=1100.0, common=null, dept=null}, Emp{empno=7902, ename='FORD', job='ANALYST', mgr=7566, hiredate=Thu Dec 03 08:00:00 CST 1981, sal=3000.0, common=null, dept=null}, Emp{empno=7934, ename='MILLER', job='CLERK', mgr=7782, hiredate=Sat Jan 23 08:00:00 CST 1982, sal=1300.0, common=null, dept=null}]
```
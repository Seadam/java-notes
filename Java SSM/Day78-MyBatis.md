# MyBatis

#### 内容摘要

MyBatis 查询配置文件，输入映射，输出映射，SQL 动态标签，多表查询，一对一，一对多，多对多，Log4J

------

#### MyBatis 查询配置文件 Mapper.xml

* 输入映射  **parameterType，最多接收一个参数**
* 输出映射 **resultType resultMap**
* **SQL** 动态标签

#### 输入映射 parameterType

简单类型，**Java** 基本类型 + **String**，见内置别名，占位符命名没有特别要求

```java
void removeUserById(String id);
User findUserByUsername(String username);
```

```xml
 <delete id="removeUserById" parameterType="string">
     delete from user where id = #{id};
</delete>
<select id="findUserByUsername" parameterType="string" 
        resultType="day77.mybatis.practice.domain.User">
    select * from user where username like #{username};
</select>
```

POJO 类型，从 POJO 里获取对应参数，要求占位符名对应 POJO **字段名** 或 **getter** 方法（推荐）

```java
void saveUser(User user);
```

```xml
<insert id="saveUser" parameterType="user">
    insert into user 
    (id, username, password, email, tel) 
    VALUES 
    (#{id}, #{username}, #{password}, #{email}, #{tel});
</insert>
```

包装 POJO 类型， 多级引用 POJO ，如 **VO （Value Object）值对象**，用于封装传递数据值

```java
public class UserVO {
    User user;
    int limit;
    // ...
}
```

```java
List<User> multiConditionSearch(UserVO userVO);
```

```xml
<select id="multiConditionSearch" parameterType="userVo" resultType="user">
    SELECT * FROM user WHERE username LIKE #{user.username} LIMIT #{limit};
</select>
```

更多级 POJO 引用

Map 类型，占位符名对应 map key

```java
User checkUser(Map<String, String> map);
```

```xml
 <select id="checkUser" parameterType="map" resultType="user">
     SELECT * FROM user WHERE username = #{username} AND password = #{password};
</select>
```

#### 输出映射 resultType

简单映射，**Java** 基本类型 + **String**

```java
int countUser();
```

```xml
<select id="countUser" resultType="int">
    SELECT COUNT(*) FROM user;
</select>
```

**POJO** 对象或 **List\<POJO\>** ，**一行数据映射到一个 POJO**

* 查询结果列名与 **POJO** 属性名要一致
* 没有字段对应， 返回 **POJO** 为 **null**
* 仅部分对应，返回填充部分属性的 **POJO**
* 查询结果列名有别名，要与别名一致，才能映射成功

#### 输出映射 resultMap

声明 **查询结果列名(column)**  与 **返回结果对象属性名(property)**  映射关系

* **用于多表查询，封装有引用的 POJO，具体见下多表查询**


* `<id >` 映射主键
* `<result>` 映射其他结果列  `<result column="列名" property="属性名"/>`

```java
public class Result {   
    String resultId;
    String resultUsername;
    String resultPassword;
    // ...
}
```

```xml
<resultMap id="resultMapByUsername" type="result">
    <id column="id" property="resultId"/>
    <result column="username" property="resultUsername"/>
    <result column="password" property="resultPassword"/>
</resultMap>
<select id="resultUserByUsername"
        parameterType="string" 
        resultMap="resultMapByUsername">
    SELECT * FROM user WHERE username = #{username};
</select>
```

#### SQL 动态标签

`<where>`  自动去掉第一个 **AND**，如果没有满足条件，会去掉 `WHERE` 

> *choose when otherwise*
>
> *where* 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句。而且，若语句的开头为“AND”或“OR”，*where* 元素也会将它们去除。 [source](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)

`<if>`

* 当传入一个参数时（如 String）要使用 **`_parameter` 引用当前参数** `<if test="_parameter != null">`
* 判断字符串是否为空以及空串 `<if test="username != null and !username.isEmpty()">`
* 成员可以调用方法`<if test="username.length() > 8">`   **OGNL** 表达式（高级版 EL 表达式）

```xml
<!-- 原始做法 拼接 where true -->
<select id="findUsersByUsername" parameterType="string" resultType="user">
    SELECT * FROM user WHERE true AND username LIKE #{username};
</select>

<!-- 传入一个参数时 -->
<select id="findUsersByUsername" parameterType="string" resultType="user">
    SELECT * FROM user
    <where>
        <if test="_parameter != null and !_parameter.isEmpty()">
            AND username LIKE #{username}
        </if>
    </where>
</select>

<!-- 传入 POJO 对象-->
<select id="findUsersByUsername" parameterType="condition" resultType="user">
    SELECT * FROM user
    <where>
        <if test="username != null and !username.isEmpty()">
            AND username LIKE #{username}
        </if>
    </where>
</select>
```

**`<sql>` SQL 片段**，多条件查询分页，先查总数，再限制查询，**where 条件都是一样的，可以复用**

* `<include refid=""/> `

```xml
<!-- 定义 -->
<sql id="multiCondition">
    <where>
        <if test="user.username != null and !user.username.isEmpty()">
            AND username LIKE #{user.username}
        </if>
        <if test="user.email != null and !user.email.isEmpty()">
            AND email LIKE #{user.email}
        </if>
        <if test="user.tel != null and !user.tel.isEmpty()">
            AND tel LIKE #{user.tel}
        </if>
    </where>
</sql>

<!-- 引用 -->
<select id="multiConditionSearchPage" parameterType="userVo" resultType="user">
    SELECT * FROM user
    <include refid="multiCondition"/>
    LIMIT #{limit}
</select>

<select id="multiConditionSearchCount" parameterType="userVo" resultType="int">
    SELECT count(*) FROM user
    <include refid="multiCondition"/>
</select>
```

`<foreach>`  可以循环传入参数值，批量删除 **checkbox id，**使用 **IN** 如 `where id IN (2,3,4)`

>  你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象传递给 *foreach* 作为集合参数。当使用可迭代对象或者数组时，index 是当前迭代的次数，item 的值是本次迭代获取的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值

```java
void deleteMulti(int[] ids);
```

```xml
<!-- 当传入的参数是数组时，可以不用指定 parameterType 使用 array 引用当前数组参数 -->
<!-- 原始写法 or id = #{id} -->
<select id="multiDelete">
    DELETE FROM user
    WHERE id IN
    <foreach collection="array" item="id"
             open="(" separator="," close=")">
        #{id}
    </foreach>
</select>

<!-- 当传入的参数是列表 list 时，可以不用指定 parameterType 使用 list 引用当前数组参数 -->
<foreach collection="list"/>

<!-- 当传入的参数是 POJO 里的集合时，需要指定 parameterType，成员名引用参数 -->
<select id="multiDelete" parameterType="userVo">
    DELETE FROM user
    WHERE id IN
    <foreach collection="ids" item="id"
             open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

`<set>` 动态更新语句，动态前置 SET 关键字，同时也会删掉无关的逗号

```xml
<update id="updateUser" parameterType="user">
    UPDATE user
    <set>
        <if test="username != null">username=#{username},</if>
        <if test="password != null">password=#{password},</if>
        <if test="tel != null">tel=#{tel},</if>
    </set>
    WHERE id = #{id}
</update>
```

`<bind>` 创建一个变量并将其绑定到上下文

```xml
<select id="findUsersByUsername" resultType="user">
  <bind name="pattern" value="'%' + user.username + '%'" />
  SELECT * FROM user
  WHERE username LIKE #{pattern}
</select>
```

### 多表查询

多表设计

* 一对一：用户——购物车
* 一对多：用户——订单项
* 多对多：用户——商品，学生——课程

#### 一对一 unique

测试表 user_ cellphone_  一对一（ **cid unique**）

```mysql
CREATE TABLE user_
(
  uid      INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(32) NOT NULL,
  cid      INT         NOT NULL,
  CONSTRAINT user__cid_uindex UNIQUE (cid),
  CONSTRAINT user__cellphone__cid_fk
  FOREIGN KEY (cid) REFERENCES cellphone_ (cid)
) ENGINE = InnoDB;

CREATE TABLE cellphone_
(
  cid   INT AUTO_INCREMENT PRIMARY KEY,
  cname VARCHAR(32) NOT NULL
) ENGINE = InnoDB;
```

Bean 类，一对一的关系，一个对象持有另一个引用

```java
class User {
    int uid;
    String username;
}
class Cellphone {
    int cid;
    String cname;
}
```

#### **多表关联查询，方法一**，扩展 User 类联合 CellPhone，查询结果自动映射到 **UserExt**

```java
class UserExt extends User {
    int cid;
    String cname;
}
```

UserMapper.java

```java
List<UserExt> findUserAndCellphone();
```

UserMapper.xml 外链接查询  **resultType**

```xml
<select id="findUserAndCellphone" resultType="userExt">
    SELECT * FROM user_ LEFT OUTER JOIN cellphone_ c ON user_.cid = c.cid;
</select>
```

#### 多表关联查询，方法二，手写 **resultMap** 指定映射接收结果集

UserMapper.xml  **resultMap**

```xml
<resultMap id="userAndCellphone" type="userExt">
        <id column="uid" property="uid"/>
        <result column="cid" property="cid"/>
        <result column="username" property="username"/>
        <result column="cname" property="cname"/>
    </resultMap>
    <select id="findUserAndCellphone" resultMap="userAndCellphone">
    SELECT * FROM user_ LEFT OUTER JOIN cellphone_ c ON user_.cid = c.cid;
</select>
```

#### 多表关联查询，方法三， \<association> 一个对象持有另一个对象引用，推荐使用！！！

单向一对一，一个对象持有另一个对象引用

```java
class User {
    int uid;
    String username;
    // 单向一对一
    Cellphone cellphone;
}
```

UserMapper.java

```java
List<User> findUserWithCellphone();
```

UserMapper.xml，手写 **resultMap** 指定映射接收结果集 **\<association>**

```xml
<resultMap id="userWithCellphone" type="user">
    <id column="uid" property="uid"/>
    <result column="username" property="username"/>
     <!-- 
	property 当前填充对象 引用类型 属性名
	javaType select 查询结果返回类型（当前引用类型）
	select 当前引用类型查询语句，namespace + id，同一 namespace 可以省略
	column 当前引用类型查询语句参数
	-->
    <association property="cellphone"
                 javaType="cellphone"
                 select="day78.mybatis.demo.mapper.UserMapper.findCellphoneByUid"
                 column="uid">
        <id column="cid" property="cid"/>
        <result column="cname" property="cname"/>
    </association>
</resultMap>

<select id="findUserWithCellphone" resultMap="userWithCellphone">
    SELECT * FROM user_
</select>

<select id="findCellphoneByUid" resultType="cellphone">
    SELECT * FROM cellphone_ WHERE cid = #{cid}
</select>
```

简化，当列名和 Bean 的字段一致时，可以省略，**主键不能省略**

> javaType 属性是不需要的,因为 MyBatis 在很多情况下会为你算出来  ofType
>
> `<collection property="posts" column="id" ofType="Post" select="selectPostsForBlog"/>`

```xml
<!-- 主键不能省略 -->
<resultMap id="userWithCellphone" type="user">
    <id column="uid" property="uid"/>
    <association property="cellphone"
                 javaType="cellphone"
                 select="day78.mybatis.demo.mapper.UserMapper.findCellphoneByUid"
                 column="uid">
        <id column="cid" property="cid"/>
    </association>
</resultMap>
```

#### 一对多  \<collection>

一个 user 对应多个 cellphone

```java
class User {
    int uid;
    String username;
    // 一对多
    List<Cellphone> cellphones;
}
```

UserMapper.java

```java
User findUserById(int id);
```

UserMapper.xml **\<collection>**

````xml
<resultMap id="userWithCellphones" type="user">
    <id column="uid" property="uid"/>
    <collection property="cellphones"
                javaType="list"
                select="day78.mybatis.demo.mapper.UserMapper.findCellphoneByUid"
                column="uid">
        <id column="cid" property="cid"/>
    </collection>
</resultMap>
    
<select id="findUserByUid" parameterType="int" resultMap="userWithCellphones">
    SELECT * FROM user_ WHERE uid = #{uid}
</select>
    
<select id="findCellphoneByUid" resultType="cellphone">
    SELECT * FROM cellphone_ WHERE uid = #{uid}
</select>
````

#### 多对多

student - relation - course，建表

```mysql
CREATE TABLE student_ (
    sid INT PRIMARY KEY AUTO_INCREMENT,
    sname VARCHAR(32)
);

CREATE TABLE course_ (
    cid INT PRIMARY KEY AUTO_INCREMENT,
    cname VARCHAR(32)
);

CREATE TABLE t_sc    (
    tid INT PRIMARY KEY AUTO_INCREMENT,
    sid INT ,
    cid INT ,
    CONSTRAINT sid_t_sc_fk FOREIGN KEY (sid) REFERENCES student_(sid),
    CONSTRAINT cid_t_sc_fk FOREIGN KEY (cid) REFERENCES course_(cid)
);
```

bean，注意 **toString()** 循环引用问题 

```java
class Student {
    int sid;
    String sname;
    // 双向一对多，就是多对多
    List<Course> courseList;
}

class Course {
    int cid;
    String cname;
    // 双向一对多，就是多对多
    List<Student> students;
}
```

StudentMapper.java

```java
Student findStudentBySid(int sid);
```

StudentMapper.xml

```xml
<resultMap id="studentAndCourses" type="student">
    <id column="sid" property="sid"/>
    <collection property="courseList"
                javaType="list"
                select="day78.mybatis.demo.mapper.StudentMapper.findCoursesBySid"
                column="sid">
        <id column="cid" property="cid"/>
    </collection>
</resultMap>

<select id="findStudentBySid" parameterType="int" resultMap="studentAndCourses">
    SELECT * FROM student_ WHERE sid = #{sid}
</select>

<select id="findCoursesBySid" parameterType="int" resultType="course">
    SELECT * FROM t_sc INNER JOIN course_ c ON t_sc.cid = c.cid WHERE sid = #{sid};
</select>
```

#### ORM 规范写法

一对一，一对多，多对多

#### Log4J

级别向上包括，**Level**： `FATAL` `ERROR` `INFO` `DEBUG` `TRACE` `ALL`， 常用配置

```properties
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n


### set log levels - for more verbose logging change 'info' to 'debug' ###
# Global logging configuration
log4j.rootLogger=all, stdout

# MyBatis logging configuration...
# log4j.logger.org.mybatis.example.BlogMapper=TRACE
 
 
 
### direct messages to file mybatis.log ###
# log4j.appender.file=org.apache.log4j.FileAppender
# log4j.appender.file.File=~/mybatis.log
# log4j.appender.file.layout=org.apache.log4j.PatternLayout
# log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n



##########################################

#log4j.logger.org.hibernate=info
#log4j.logger.org.hibernate=debug

### log HQL query parser activity
#log4j.logger.org.hibernate.hql.ast.AST=debug

### log just the SQL
#log4j.logger.org.hibernate.SQL=debug

### log JDBC bind parameters ###
#log4j.logger.org.hibernate.type=info
#log4j.logger.org.hibernate.type=debug

### log schema export/update ###
#log4j.logger.org.hibernate.tool.hbm2ddl=debug

### log HQL parse trees
#log4j.logger.org.hibernate.hql=debug

### log cache activity ###
#log4j.logger.org.hibernate.cache=debug

### log transaction activity
#log4j.logger.org.hibernate.transaction=debug

### log JDBC resource acquisition
#log4j.logger.org.hibernate.jdbc=debug

### enable the following line if you want to track down connection ###
### leakages when using DriverManagerConnectionProvider ###
#log4j.logger.org.hibernate.connection.DriverManagerConnectionProvider=trace

```










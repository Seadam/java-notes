# MyBatis

#### 内容摘要

fremarker 介绍，入门案例，基本语法，Spring MVC 使用 Freemarker，多视图解析器，MyBatis 介绍，入门，针对单表进行增删改查，问题

---

### Freemarker 介绍

页面静态化技术，模版引擎，模版输出静态 HTML ／电子邮件／XML ／源码等

使用场景

* HRM *Human resource management*  人力资源管理
* ERP *enterprise resource planning* 企业资源计划
* CMS *Content Management System* 内容管理系统，新闻／官网／组织／政府

#### Freemarker 入门案例

导包

* `freemarker.jar` 

模版  index.ftl

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
<h1>Hello ${name}</h1>
<a href="${url}">${message}</a>
</body>
</html>
```

产生  `freemarker.template.Confituration`

```java
// 1. 创建一个 Configuration 对象，构造方法传入当前版本
Configuration configuration = new Configuration(Configuration.getVersion());
// 2. 设置模版文件所在目录
configuration.setDirectoryForTemplateLoading(new File("template"));
// 3. 设置模版使用字符集
configuration.setDefaultEncoding("utf-8");
// 4. 加载模版，创建模版对象  template/index.ftl
Template template = configuration.getTemplate("index.ftl");
// 5. 创建模版使用的数据集，仅支持 POJO、Map，集合等需要先放到 map 里面
Map<String, String> data = new HashMap<>();
data.put("name", "Tom");
data.put("url", "#");
data.put("message", "Welcome Freemarker");
// 6. 创建 FileWriter，模版生成后写入到文件
FileWriter fileWriter = new FileWriter("output/index.html");
// 7. 调用模版输出
template.process(data, fileWriter);
// 8. 关闭流
fileWriter.close();
```

#### Freemarker VS Jsp

相同点

* 通过模版动态生成页面
* 支持表达式语言

不同点

* Jsp 是 Java 类，使用时会创建对象
* Freemarker 是模版技术，如果仅访问模版化的静态页面，不会占用 JVM 内存

优势

* 不需要 Web 容器，J2SE 也可以使用
* 一次生成的静态页面可以重复使用

#### 基本语法

* 拿到 Map 中的值，同上
* 拿到 POJO 中的属性

```java
// 如果将 POJO 放入 map
User user = new User("Jerry", "10086", 22);
data.put("user", user);
// 模版里拿要加前缀
<h1>Hello ${user.name}</h1>
<h2>Tel: ${user.tel}</h2>
<h2>Age: ${user.age}</h2>

// 如果直接放入 POJO
User data = new User("Jerry", "10086", 22);
// 模版里根据属性名可以直接拿出属性值
<h1>Hello ${name}</h1>
<h2>Tel: ${tel}</h2>
<h2>Age: ${age}</h2>
```

* 遍历集合    <# list 中间没有空格>

```html
<!--  List<User> list = Arrays.asList(users);
        data.put("users", list); -->
<#list users as user>
<h1>Hello ${user.name}</h1>
<h2>Tel: ${user.tel}</h2>
<h2>Age: ${user.age}</h2>
</#list>
```

* 取集合中下标

```html
<#list users as user>
index: ${user_index}
<!-- ... -->
</#list>
```

* 条件判断，比较字符串用 `==` 或 `=` `<#if> <#elseif> <#else> </#if>`

```html
<h1>Hello ${user.name}
    <#if user.name == "Tom">I am Tom
    <#elseif user.name == "Jerry">I am Jerry
    <#else>I am Ken
    </#if>
</h1>
```

* 特殊值处理
  * **null** 处理
    * `<#if user??>` 判断 **user** 是否为 **null**
    * 如果取到的值为 **null** ，或者没有这个**键值对**，会抛异常
      * 解决：在该属性后加 `!`，若该值为 **null**，或不存在时，返回空串
      * 如 `${age!}`
    * 当该值为 **null** 时，设置显示的默认值 `${name! "admin"}`
  * 日期处理，自定义日期转换  `Date`  =>  `?string(format)`

```html
<!-- 判断该值是否为空 -->
<#if name??></#if>
<!-- 值为 null ，或者没有这个键值对 -->
${age!}
<!-- 如果该值为 null，或者不存在时，默认值 -->
${name! "admin"}
<!-- 日期处理 -->
${user.birthday?string("yyyy-MM-dd HH:mm")}
```

* **include** 引入其他静态文件或模版

```html
<#include "footer.ftl"/>
```

#### Spring MVC 使用 Freemarker

导包

* `freemarker.jar` 
* `spring-context-support-5.0.4.RELEASE.jar`

模版

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>freemarker</title>
</head>
<body>
<h1>Hello Freemarker</h1>
<h2>username: ${user.username}</h2>
<h2>password: ${user.password}</h2>
</body>
</html>
```

**webContext.xml** 配置 **Freemarker Configuration**

```xml
<!-- freemarkerConfiguration -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/ftl"/>
    <property name="defaultEncoding" value="utf-8"/>
</bean>
```

或者

```xml
<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/WEB-INF/ftl"/>
</mvc:freemarker-configurer>
```

**webContext.xml** 配置模版结果视图解析器

```xml
<!-- freemarkerViewResolver -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="contentType" value="text/html;charset=utf-8"/>
    <property name="suffix" value=".ftl"/>
</bean>
```

目录结构

```java
web
 |- WEB-INF
    |- ftl
        |- hello.ftl
    |- jsp
   		|- index.jsp
```

给模版数据

```java
// Model model 就是一个 map 
// /WEB-INF/ftl/hello.ftl
@RequestMapping("/freemarker")
    public String freemarker(Model model) {
    model.addAttribute("user", new User("root", "root"));
    return "hello";
}
```

Spring MVC 直接将渲染后的模版页面，写入 **response** 流，不会存储在服务器

**Srping MVC** 与 **Freemarker** 结合仅是使用了模版技术，静态化需要自己手动实现

#### 多视图解析器

使用 **Freemarker** 时需要访问 **jsp** 或其他页面，还需要配置 `InternalResourceResolver`

```xml
<!-- 默认页面解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

解法一：配置优先级，**order 是顺序**

* 先找合适的视图解析器，如果没找到，找 **order 顺序** 下一个解析器
* 遇到页面解析时，会先去匹配第一个，然后匹配第二个
* 先找 `.ftl`  在找  `.jsp`  
* 缺点，按顺序匹配，不如解法二好

```xml
<!-- return "show"/>
<!-- freemarkerViewResolver -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <!-- <property name="templateLoaderPath" value="/WEB-INF/ftl"/> -->
    <!-- 不能加 prefix ，使用 templateLoaderPath 前缀  -->
    <property name="suffix" value=".ftl"/>
    <property name="order" value="1"/>
</bean>
<!-- 默认页面解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp"/>
    <property name="suffix" value=".jsp"/>
    <property name="order" value="2"/>
</bean>
```

解法二：根据视图名前缀，直接去找合适的视图解析器

```xml
<!-- return "jsp/index" -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 此处前缀要是 /WEB-INF/ -->
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
    <property name="viewNames" value="jsp*"/>
</bean>

<!-- return "ftl/user" -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <!-- <property name="templateLoaderPath" value="/WEB-INF/"/> -->
    <!-- 此处前缀要是 /WEB-INF/ -->
    <property name="suffix" value=".ftl"/>
    <property name="viewNames" value="ftl*"/>
    <property name="contentType" value="text/html;charset=utf-8"/>
</bean>
```

### MyBatis

#### MyBatis 介绍

历史发展

* JDBC => Hibername => iBatis => MyBatis

ORM 框架 *Object Relationship Mapping* 对象关系映射

* MyBatis 封装了 JDBC 持久层框架
* 暴露 SQL 语句，修改方便，其他封装起来

JPA 规范  *Java Persistence API* Java 持久化规范

#### 入门

导包

* `mybatis-3.4.4.jar`
* jdbc
* `/lib`

建测试数据库，建表，对应 Java Bean

```sql
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(32),
  password VARCHAR(32),
  email VARCHAR(32),
  tel INT
);
```

创建 MyBatis 主配置文件 **src/mybatis.xml**  配置数据库连接环境信息 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <!-- 环境配置 -->
        <environment id="development">
            <!-- 事务管理交给 mybatis -->
            <transactionManager type="JDBC"/>
        d    <!-- 连接池 POOLED 默认连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root123456"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

创建 MyBatis SQL 语句配置文件  一个 Bean 对应一个配置文件， **bean/User.xml**

* namespace 该映射 命名空间
* id 定位 SQL 语句 
* **parameterType** 查询参数类型
* **resultType** 返回值类型，全类名，返回每一行映射的对象
* `#{id}`  占位符，别名，当传入 **POJO** 时，占位符要对应 **bean** 类成员变量名，还是 get/set
  * 只有一个参数需要传递，命名随意s

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="User">
    <select id="findUserById" parameterType="int"  
            resultType="day76.mybatis.demo.bean.User">
        select * from user where id = #{id};
    </select>
</mapper>
```

**mybatis.xml** 引入 **User.xml**

```xml
<mappers>
    <mapper resource="day76/mybatis/demo/bean/User.xml"/>
</mappers>
```

**Java 代码，查询**

```java
public void findUserById() throws IOException {
    // 加载配置文件 org.apache.ibatis.io.Resources
    InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

    // 打开 SQL 连接会话，操作数据库用
    SqlSession sqlSession = factory.openSession();

    // 指定查询的 sql 语句， 对应配置文件 Mapping namespace id
    User user = sqlSession.selectOne("User.findUserById", 2);

    System.out.println("user = " + user);

    // 关闭会话
    sqlSession.close();
}
```

#### 针对单表进行增删改查

```java
// 测试环境
@Before
public void before() throws IOException {
    // 加载配置文件 org.apache.ibatis.io.Resources
    InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
    // 打开 SQL 连接会话，操作数据库用
    sqlSession = factory.openSession();
}

@After
public void after() {
    // 关闭会话
    sqlSession.close();
}
```

查

```java
// 查，模糊查询，用户名，返回符合的 list
@Test
public void findUserByUsername() {
    List<User> users = sqlSession.selectList("User.findUserByUsername", "%o%");
    System.out.println("users = " + users);
}

// 查，查询 UserList
@Test
public void listUser() {
    List<User> users = sqlSession.selectList("User.listUser");
    System.out.println("users = " + users);
}

// 查，查询 User 总数
@Test
public void countUser() {
    int count = sqlSession.selectOne("User.countUser");
    System.out.println("count = " + count);
}

// 查，查询一个 User
@Test
public void findUserById() {
    // 指定查询的 sql 语句， 对应配置文件 Mapping namespace id
    User user = sqlSession.selectOne("User.findUserById", 2);
    System.out.println("user = " + user);
}

// 查，用户登录，用户名和密码
@Test
public void checkUser() {
    Map<String, String> map = new HashMap<>();
    map.put("username", "root");
    map.put("password", "root");
    User user = sqlSession.selectOne("User.checkUser", map);
    System.out.println(user);
}
```

```xml
<mapper namespace="User">
    <select id="checkUser" parameterType="java.util.Map" 
            resultType="day76.mybatis.demo.bean.User" >
        select * from user where username = #{username} and password = #{password};
    </select>
    <select id="findUserById" parameterType="int" 
            resultType="day76.mybatis.demo.bean.User">
        select * from user where id = #{id};
    </select>
    <select id="countUser" resultType="int">
        select count(*) from user;
    </select>
    <select id="listUser" resultType="day76.mybatis.demo.bean.User">
        select * from user;
    </select>
    <!-- string 小写，表示 Java String 类型 java.lang.Stirng 简写 -->
    <select id="findUserByUsername" parameterType="string" 
            resultType="day76.mybatis.demo.bean.User">
        select * from user where username like #{username};
    </select>
</mapper>
```

增／删／改，**自动开启事务，要手动提交**

```java
// 增，添加 User
@Test
public void saveUser() {
    // User user
    sqlSession.insert("User.saveUser", user);
    // 增删改自动开启事务，需要手动提交
    sqlSession.commit();
}
```

```xml
<insert id="saveUser" parameterType="day76.mybatis.demo.bean.User">
    insert into user (username, password, email, tel) 
    VALUES (#{username}, #{password}, #{email}, #{tel});
</insert>
```

```java
// 改，修改 User
@Test
public void updateUser() {
    // User user ...
    int update = sqlSession.update("User.updateUser", user);
    // 增删改自动开启事务，需要手动提交
    sqlSession.commit();
}
```

```xml
<update id="updateUser" parameterType="day76.mybatis.demo.bean.User">
    update user set username = #{username}, password = #{password}, 
    email = #{email}, tel = #{tel} where id = #{id};
</update>
```

```java
// 删，根据 id 删 User
@Test
public void removeUserById() {
    int delete = sqlSession.delete("User.removeUserById", 3);
    // 增删改自动开启事务，需要手动提交
    sqlSession.commit();
}
```

```xml
<delete id="removeUserById" parameterType="int">
    delete from user where id = #{id};
</delete> 
```

#### 问题

如果主键是 UUID ，传统方式

```java
// 如果主键是 UUID 的插入操作
@Test
public void insertUserManualUUID() {
    User user = new User();
    // 手动设置 UUID
    user.setId(UUID.randomUUID().toString());
    int insert = sqlSession.insert("User.insertUserManualUUID", user);
    sqlSession.commit();
}
```

```xml
<insert id="insertUserManualUUID" parameterType="day76.mybatis.demo.bean.User">
    insert into user (id, username, password, email, tel) 
    VALUES (#{id}, #{username}, #{password}, #{email}, #{tel});
</insert>
```

MyBatis 解决方案，使用 **selectKey** 预处理设置 **UUID**

```java
// 如果主键是 UUID 自动生成操作
@Test
public void insertUserAutoUUID() {
    User user = new User();
    user.setId("123"); // 不会起作用
    int insert = sqlSession.insert("User.insertUserAutoUUID", user);
    sqlSession.commit();
    System.out.println("insert = " + insert);
}
```

```xml
<insert id="insertUserAutoUUID" parameterType="day76.mybatis.demo.bean.User">
    <selectKey keyProperty="id" resultType="string" order="BEFORE">
        select UUID();
    </selectKey>
    insert into user (id, username, password, email, tel) 
    VALUES (#{id}, #{username}, #{password}, #{email}, #{tel});
</insert>
```

返回上一个插入的 ID，仅支持 mysql 

```sql
SELECT LAST_INSERT_ID();
```

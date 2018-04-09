# MyBatis

#### 内容摘要

传统 DAO 实现，使用 MyBatis 实现 ORM DAO 层 => Mapper，主配置文件配置 mybatis.xml

---

#### 传统 DAO 实现 SqlSession 

```java
public interface UserDao {
    User findUserByUsername(String username);
    void saveUser(User user);
    void removeUserById(String id);
    List<User> listUser();
}
public class UserDaoImpl implements UserDao {

    private SqlSessionFactory factory;
   
    public void setFactory(SqlSessionFactory factory) {
        this.factory = factory;
    }

  	// 重复代码太多
    @Override
    public User findUserByUsername(String username) {
        SqlSession sqlSession = factory.openSession();
        User user = sqlSession.selectOne("User.findUserByUsername", username);
        sqlSession.close();
        return user;
    }
	// 重复代码太多
    @Override
    public void saveUser(User user) {
        SqlSession sqlSession = factory.openSession();
        sqlSession.insert("User.saveUser", user);
        sqlSession.commit();
        sqlSession.close();
    }
	// 重复代码太多
    @Override
    public void removeUserById(int id) {
        SqlSession sqlSession = factory.openSession();
        sqlSession.update("User.removeUserById", id);
        sqlSession.commit();
        sqlSession.close();
    }
	// 重复代码太多
    @Override
    public List<User> listUser() {
        SqlSession sqlSession = factory.openSession();
        List<User> users = sqlSession.selectList("User.listUser");
        sqlSession.close();
        return users;
    }
}
```

#### 使用 MyBatis 实现 ORM DAO 层 => Mapper

**Mapper** (DAO interface) 代理实现，需要遵循如下规范

* **UserMapper** 的**全类名**要与映射文件 **UserMapper.xml** 的 **namespace** 一致
* **UserMapper** 的**方法名**要与映射文件 **UserMapper.xml** 的 **id**  一致
* **UserMapper** 的**参数类型**要与映射文件 **UserMapper.xml** 的 **parameterType** 一致，最多接收一个参数
  * 使用 **Map<String, String> **可以解决传入多个参数
* **UserMapper** 的**返回值类型**要与映射文件 **User.xml** 的 **resultType** 一致
  * 如果是列表的话，只需要指定**范型类型**  `List<User>  =>  User`

创建 **Mapper** 接口，就是 **UserDao**

```java
// Mapper
public interface UserMapper {
    User findUserByUsername(String username);
    void saveUser(User user);
    void removeUserById(String id);
    List<User> listUser();
    User checkUser(Map<String, String> map);
}
```

配置 **Mapper** 配置文件，就是 **User.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="day77.mybatis.practice.mapper.UserMapper">

    <select id="checkUser" parameterType="map" 
            resultType="day77.mybatis.practice.domain.User">
        select * from user where username = #{username} and password = #{password};
    </select>
    
    <delete id="removeUserById" parameterType="string">
        delete from user where id = #{id};
    </delete>

    <insert id="saveUser" parameterType="day77.mybatis.practice.domain.User">
        insert into user 
        (id, username, password, email, tel) 
        VALUES 
        (#{id}, #{username}, #{password}, #{email}, #{tel});
    </insert>

    <select id="listUser" resultType="day77.mybatis.practice.domain.User">
        select * from user;
    </select>
    <select id="findUserByUsername" parameterType="string" 
            resultType="day77.mybatis.practice.domain.User">
        select * from user where username like #{username};
    </select>
</mapper>
```

拿到 **Mapper** 实现类，使用

```java
// SqlSession sqlSession = factory.openSession();
// UserMapper mapper = sqlSession.getMapper(UserMapper.class);
public class UserMapperTest {

    private SqlSessionFactory factory;
    private SqlSession sqlSession;
    private UserMapper mapper;

    @Before
    public void before() throws IOException {
        InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");
        factory = new SqlSessionFactoryBuilder().build(inputStream);
        sqlSession = factory.openSession();
        mapper = sqlSession.getMapper(UserMapper.class);
    }

    @After
    public void after() {
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void listUser() {
        List<User> users = mapper.listUser();
        System.out.println(users);
    }

    @Test
    public void findUserByUsername() {
        User admin = mapper.findUserByUsername("admin");
        System.out.println("admin = " + admin);
    }

    @Test
    public void saveUser() {
        User user = new User();
        user.setUsername("Mapper");
        user.setPassword("四大");
        user.setEmail("Ben@ben.com");
        user.setTel(23333);
        user.setId("123");
        mapper.saveUser(user);
    }

    @Test
    public void removeUserById() {
        mapper.removeUserById("123");
    }

    @Test
    public void checkUser() {
        Map<String, String> map = new HashMap<>();
        map.put("username", "root");
        map.put("password", "root");
        User user = mapper.checkUser(map);
        System.out.println(user);
    }
}

```

#### 主配置文件配置 mybatis.xml

配置层次结构

```xml
<configuration 配置>
    <properties 属性/>
    <settings 设置/>
    <typeAliases 类型别名/>
    <typeHandlers 类型处理器/>
    <objectFactory 对象工厂/>
    <plugins 插件/>
    <environments 环境>
        <environment 环境变量/>
        <transactionManager 事务管理器/>
        <dataSource 数据源/>
        <databaseIdProvider 数据库厂商标识/>
    </environments>
    <mappers 映射器/>
</configuration>
```

**properties**

```xml
<properties>
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false"/>
    <property name="username" value="root"/>
    <property name="password" value="root123456"/>
</properties>
```

**jdbc.properties**

```properties
! src/properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=false
username=root
password=root123456
```

```xml
<properties resource="jdbc.properties"/>
<!-- ... -->
<dataSource type="POOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
</dataSource>
```

**settings**  设置缓存等，详见参考文档

**typeAliases** 类型取别名，常用内置别名，可以指定包名

* `map => Map` 
* `string => String`
* `int => Integer`

```xml
<typeAliases>
    <typeAlias type="day77.mybatis.practice.domain.User" alias="user"/>
</typeAliases>
<!-- parameterType="user" -->

<!-- 也可以指定包名，包中的 Java Bean，
在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名 -->
<typeAliases>
    <package name="day77.mybatis.practice.domain"/>
</typeAliases>
```

**environmental** 

```xml
<environments default="development">
    <environment id="development" 环境变量>
        <transactionManager type="JDBC" 事务处理器/>
        <dataSource type="POOLED" 数据源>
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
           <!-- 配置连接池 <property name="poolMaximumActiveConnections" value="20"/> -->
        </dataSource>
    </environment>
</environments>
```

**mappers**

```xml
<mappers>
    <!-- 通过 User.xml 去找，相对位置 --> 
    <mapper resource="day77/mybatis/practice/domain/User.xml"/>
    
    <!-- 通过 UserMapper 接口全类名去找 UserMapper.xml -->
    <!-- 映射文件名要同接口名一致，并在同一包下  -->
    <mapper class="day77.mybatis.practice.mapper.UserMapper"/>
</mappers>
```

**mappers** 可以通过包名去找，前提

* 通过 **UserMapper** 接口全类名去找 **UserMapper.xml**
* 映射文件名要同接口名一致，并在同一包下

```xml
<mappers>
    <package name="day77.mybatis.practice.mapper"/>
</mappers>
```

#### 常用配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <typeAliases>
        <package name="org.mybatis.domain"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="org.mybatis.mapper"/>
    </mappers>
</configuration>
```




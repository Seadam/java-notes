# Spring JDBC Template

#### 内容摘要

JDBC Template，XML 配置，优化，注解配置，API，对应 QueryRunner

---

#### JDBC Template

类似 **DBUtils**，对数据库操作 **JDBC** 进行模版化封装，开发者只关心 **sql** 语句，依赖于 数据源（连接池）

导包

* 4 + 1 + 1 core、expression、context、bean、aop、logging
* 驱动 `mysql-connector-java-5.1.44-bin.jar`
* 连接池 `c3p0-0.9.1.2-jdk1.3.jar`
* Spring 支持 `spring-jdbc-5.0.4.RELEASE.jar`

#### XML 配置

`core.JDBCTemplate`

**C3P0 DataSource** 的配置文件交给 **Spring** 管理

注入

```xml
<bean id="userService" class="day71.spring.demo.service.impl.UserServiceImpl">
    <property name="userDao" ref="userDao"/>
</bean>
<bean id="userDao" class="day71.spring.demo.dao.impl.UserDaoImpl">
    <property name="jdbcTemplate" ref="jdbcTemplate"/>
</bean>

<!-- 配置 JDBC Template -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource"/>
</bean>

<!-- 配置 datasource 注入 C3P0 配置文件 --> 
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/jdbc?useSSL=false"/>
    <property name="user" value="root"/>
    <property name="password" value="root123456"/>
</bean>
```

JdbcTemplate

```java
public class UserDaoImpl implements UserDao {
    private JdbcTemplate jdbcTemplate;
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    @Override
    public User findUserByUsername(String username) {
        String sql = "select * from user where username = ?";
        return jdbcTemplate.query(sql, new ResultSetExtractor<User>() {
            @Override
            public User extractData(ResultSet resultSet) 
                throws SQLException, DataAccessException {
                User user = new User();
                if (resultSet.next()) {
                    String username = resultSet.getString("username");
                    String password = resultSet.getString("password");
                    user.setUsername(username);
                    user.setPassword(password);
                }
                return user;
            }
        }, username);
    }
}
```

#### 优化

`abstract class JdbcDaoSupport JdbcDaoSuppurt`  `getJdbcTemplate()`

```java
public class UserDaoImpl extends JdbcDaoSupport implements UserDao {
    @Override
    public User findUserByUsername(String username) {
        String sql = "select * from user where username = ?";
        return getJdbcTemplate().query(sql, new ResultSetExtractor<User>() {
           ...
        }, username);
    }
}
```

```xml
<bean id="userDao" class="day71.spring.demo.dao.impl.UserDaoImpl">
    <property name="dataSource" ref="dataSource"/>
</bean>
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/jdbc?useSSL=false"/>
    <property name="user" value="root"/>
    <property name="password" value="root123456"/>
</bean>
```

#### 注解配置

```xml
<!-- 导入的 jar 包，别人的类只能使用 xml 配置 -->
<!-- C3P0 连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/jdbc?useSSL=false"/>
    <property name="user" value="root"/>
    <property name="password" value="root123456"/>
</bean>

<!-- DBCP 连接池 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/jdbc?useSSL=false"/>
    <property name="username" value="root"/>
    <property name="password" value="root123456"/>
</bean>
```

```xml
<!-- 也可以使用 properties + xml 配置 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

```java
@Service("userService")
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;  
    @Override
    public User findUser(String username) {
        return userDao.findUserByUsername(username);
    }
}
@Repository
public class UserDaoImpl extends JdbcDaoSupport implements UserDao {
    @Autowired
    public UserDaoImpl(DataSource dataSource) {
        setDataSource(dataSource);
    }
    @Override
    public User findUserByUsername(String username) {
        ...
    }
}
```

#### API

`execute`

`update` `batchUpdate` 直接返回 **int** 很方便

`query` `queryForObject`  Map\List\

`call` 执行存储过程，函数

`queryForObject(String sql, Class aClass, Object.. args)`

* **aClass** 返回对象类型，仅返回一行或一列数据，仅支持原始数据类型， **String** 类型

* 如果要返回 **Bean** 类，需要实现接口 **RowMapper** 自己封装

* > to be a single row/single column query

实现接口 `RowMapper` 可以自定义封装数据

* `public User mapRow(ResultSet rs, int rowNum)`  **rs** 已经指向每一条数据，不用调用 **next**

	> This method should not call `next()` on the ResultSet
	>  it is only supposed to map values of the current row.

#### 对应 QueryRunner

**BeanPropertyRowMapper ** ==> **BeanHandler**／**BeanListHandler**

* ` queryForObject(sql, new BeanPropertyRowMapper<>(User.class), username, password);`
  * 没找到会抛异常 **EmptyResultDataAccessException**
* `queryRunner.query(sql, new BeanHandler(User.class), username, password);`


* `getJdbcTemplate().query(sql, new BeanPropertyRowMapper<>(User.class));`
* `queryRunner.query(sql, new BeanListHandler<>(User.class));`

**queryForObject** ==> **ScalarHandler**

* `getJdbcTemplate().queryForObject(sql, String.class, username);`
* `(Long)queryRunner.query(sql, new ScalarHandler(), username);`
* ` queryForObject(sql, new BeanPropertyRowMapper<>(User.class), username, password);`

```java
// row count
int rowCount = jdbcTemplate.queryForObject("select count(*) from user", Integer.class);

// 如果可以，尽量自己手写填充类， BeanPropertyRowMapper 牺牲性能
public List<Actor> findAllActors() {
    return this.jdbcTemplate.query( "select first_name, last_name from t_actor", new ActorMapper());
}
private static final class ActorMapper implements RowMapper<Actor> {
    public Actor mapRow(ResultSet rs, int rowNum) throws SQLException {
        Actor actor = new Actor();
        actor.setFirstName(rs.getString("first_name"));
        actor.setLastName(rs.getString("last_name"));
        return actor;
    }
}
```

```java
// 拿到返回的自增 id
 public Integer saveStudent(final Student student) {
     final KeyHolder keyHolder = new GeneratedKeyHolder();
     final String sql = "insert into student (id, name, chinese, math, english)" +
         "values (null, ?, ?, ?, ?)";
     getJdbcTemplate().update(new PreparedStatementCreator() {
         @Override
         public PreparedStatement createPreparedStatement(Connection connection) 
             throws SQLException {
             PreparedStatement preparedStatement = 
                 connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
             	// connection.prepareStatement(sql, new String[]{"id"});
             preparedStatement.setString(1, student.getName());
             preparedStatement.setDouble(2, student.getChinese());
             preparedStatement.setDouble(3, student.getMath());
             preparedStatement.setDouble(4, student.getEnglish());
             return preparedStatement;
         }
     }, keyHolder);
     return keyHolder.getKey().intValue();
 }
```






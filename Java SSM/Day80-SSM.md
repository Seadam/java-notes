# Spring + SpringMVC + MyBatis

#### 内容摘要

Spring + MyBatis，MyBatis，Spring，Mapper 注入，Spring + SpringMVC

------

### Spring + MyBatis

#### MyBatis

测试案例

```java
假设需求
- 用户注册
- 用户登录
- 登录回显

dao 层
- 插入用户
- 查询用户是否已注册
- 根据账户密码查用户
- 根据 id 查用户所有信息
```

数据表

```mysql
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT ,
  username VARCHAR(32) NOT NULL ,
  password VARCHAR(32) NOT NULL ,
  email VARCHAR(32) NOT NULL
);
```

搭架子 目录结构

```java
Spring-SpringMVC-MyBatis
	|- .idea
	|- resources
	|- src
		|- org.ssm
				|- domain
				|- mapper
				|- service
					|- impl
                |- controller
				|- test
	|- web
		|- WEB-INF
			|- classes
			|- lib
			|- resources
			|- web.xml
```

bean 

```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private String email;
}
```

mapper

```java
public interface UserMapper {
    int insert(User record);
    User selectByPrimaryKey(Integer id);
    int updateByPrimaryKeySelective(User record);
}
```

service

```java
public interface UserService {
    User login(User user);
    boolean register(User user);
    User findUserById(int id);
    boolean isUsernameExist(String username);
}
```

MyBatis 导包

* 先导 MyBatis，避免包冲突
* JDBC 驱动

配置文件

* 优先创建 **web.xml** 自动生成
* mybatis.xml，可以省略，由 **Spring** 完全接管

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="org.ssm.domain"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" 
                          value="jdbc:mysql://localhost:3306/ssm?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="root123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name="org.ssm.mapper"/>
    </mappers>
</configuration>
```

* jdbc.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm?useSSL=false
jdbc.username=root
jdbc.password=root123456
```

* log4j.properties

```properties
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

# Global logging configuration
log4j.rootLogger=all, stdout
```

##### 测试 Mybatis mapper

#### Spring

Spring 导包

* 基础：core, context, expression, aop, bean（不用 commons-logging）
* 切面：aspects, aopalliance, weaver
* 数据库：jdbc, tx

**spring.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd 
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">
</beans>
```

#### Mapper 注入问题，原始解法

注入 SqlSessionFactory

```java
@Repository
public class UserMapperImpl implements UserMapper {
    @Autowired
    SqlSessionFactory sqlSessionFactory;

    @Override
    public User selectByPrimaryKey(Integer id) {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectByPrimaryKey(1);
    }
}
```

##### 导包 MyBatis + Spring

* `mybatis-spring.jar`
* 连接池  C3P0

**spring.xml** 配置 **SqlSessionFactoryBean**

```xml
<!-- 注解包扫描 -->
<context:component-scan base-package="org.ssm"/>
<!-- 数据库配置文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!-- SqlSessionFactoryBean -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis.xml"/>
    <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 连接池交给 Spring 管理，事务管理器要求是同一个 dataSource -->
<!-- C3P0 数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

**mybatis.xml** 更新配置

```xml
<!-- TransactionManager DataSourece 由 Spring 管理 -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="org.ssm.domain"/>
    </typeAliases>
    <mappers>
        <package name="org.ssm.mapper"/>
    </mappers>
</configuration>
```

#### Mapper 注入问题 Spring 实现注入 mapper 

**spring.xml** 配置 **SqlSessionFactoryBean**，**MapperFactoryBean**

* 先获取 sqlSession
* 根据接口获取 Mapper 实现
* 实现自动注入 Mapper

```xml
<!-- SqlSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis.xml"/>
    <property name="dataSource" ref="dataSource"/>
</bean>

<!-- MapperFactoryBean -->
<bean class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <property name="mapperInterface" value="org.ssm.mapper.UserMapper"/>
</bean>
```

```xml
<!-- SqlSessionFactoryBean，直接在 Spring 配置，不需要 mybatis.xml -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="typeAliasesPackage" value="org.ssm.domain"/>
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 配置扫描 Mapper 接口包，动态实现 Mapper 接口 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 注入 sqlSessionFactory -->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    <!-- 扫描 Mapper 接口包 -->
    <property name="basePackage" value="org.ssm.mapper" />
</bean>
```

**mybatis.xml** 更新，更简化

```xml
<configuration>
    <typeAliases>
        <package name="org.ssm.domain"/>
    </typeAliases>
</configuration>
```

### Spring + SpringMVC

导包

* mvc：web, webmvc

**spring.xml** 更新配置

```xml
<!-- SpringMVC 注解驱动 -->
<mvc:annotation-driven/>
```

**web.xml** 配置 **DispatcherServlet** 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="false">
    <servlet>
        <servlet-name>springMVCServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>        
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>   
        <servlet-name>springMVCServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

**spring.xml** 配置页面处理器，将 jsp 页面放到 WEB-INF/jsp 下

```xml
<!-- Jsp 页面处理器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

#### Spring 管理 MyBatis 事务

mybatis 增删改默认开启事务，要让 Spring 管理事务 `@Transactional` ，同一个 **DataSource**

**spring-config.xml** 更新配置

```xml
<!-- Spring 事务驱动 -->
<tx:annotation-driven/>
<!-- 事务管理器 -->
<bean id="transactionManager" 
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

#### Spring  JUnit Test

导包

* `spring-test-5.0.4.RELEASE.jar`

使用

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring.xml")
public class UserServiceTest {  
    @Autowired
    UserService service;
    
    public void findUserById() {
        User user = service.findUserById(1);
        System.out.println(user);
    }
}
```

#### Jar

```java
ant-1.9.6.jar
ant-launcher-1.9.6.jar
asm-5.2.jar
aspectjweaver-1.8.13.jar
c3p0-0.9.1.2-jdk1.3.jar
cglib-3.2.5.jar
com.springsource.org.aopalliance-1.0.0.jar
commons-logging-1.2.jar
javassist-3.21.0-GA.jar
log4j-1.2.17.jar
log4j-api-2.3.jar
log4j-core-2.3.jar
mybatis-3.4.4.jar
mybatis-spring-1.3.0.jar
mysql-connector-java-5.1.44-bin.jar
ognl-3.1.14.jar
slf4j-api-1.7.25.jar
slf4j-log4j12-1.7.25.jar
spring-aop-5.0.4.RELEASE.jar
spring-aspects-5.0.4.RELEASE.jar
spring-beans-5.0.4.RELEASE.jar
spring-context-5.0.4.RELEASE.jar
spring-core-5.0.4.RELEASE.jar
spring-expression-5.0.4.RELEASE.jar
spring-jdbc-5.0.4.RELEASE.jar
spring-test-5.0.4.RELEASE.jar
spring-tx-5.0.4.RELEASE.jar
spring-web-5.0.4.RELEASE.jar
spring-webmvc-5.0.4.RELEASE.jar
```






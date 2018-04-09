# Spring IoC&DI

#### 内容摘要

Spring 简介，IoC 控制反转，DI 依赖注入，ApplicationContext，基于 XML 装配 Bean，Bean 作用域，生命周期，后处理，属性的依赖注入，集合注入，注解配置，注解API

---

#### Spring 简介

**Spring** 是一个分层的 Java EE / SE 一站式轻量级开源框架

**Spring** 核心

* **IoC Inversion of Control** 控制反转
* **AOP Aspect-oriented Programming** 面向切面编程

**Spring** 优点

* 方便解耦，简化开发（高内聚低耦合）
  * Bean 工厂，管理所有对象的创建和依赖
* AOP 编程支持
  * 面向切面编程，**增强**
  * 权限拦截，运行监控等
* 声明式事务的支持
  * 通过配置文件配置事务
* 方便的程序测试
  * 支持 Junit 4
* 方便集成各种优秀框架
  * Struts2、Hibernate、MyBatis
* 降低 JavaEE API 使用难度
  * 封装 JDBC、JavaMail、远程调用等

#### IoC *Inversion of Control*

控制反转，控制对象的生成交给 **Spring** 管理，解耦

搭建 IoC 项目

* 导包（4 + 1）

  * `com.springsource.org.apache.commons.logging-1.1.1.jar`
  * `spring-beans-5.0.4.RELEASE.jar` 
  * `spring-context-5.0.4.RELEASE.jar` 
  * `spring-core-5.0.4.RELEASE.jar` 
  * `spring-expression-5.0.4.RELEASE.jar`
* 新建类
  * server
  * dao
* 配置文件

  * 新建 `src` 目录下
  * 配置文件名 `applicationContext.xml`
  * 约束： `Docs / index.html / Core / appendix`  去找
  * `ClassPathXmlApplicationContext` 加载 `src` 目录下配置文件

```xml
<!-- applicationContext.xml 示例 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 	
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="foo" class="x.y.Foo">
        <meta key="cacheName" value="foo"/>
        <property name="name" value="Rick"/>
    </bean>

</beans>
```

```xml
<!-- 配置 applicationContext.xml -->
<bean id="userServiceImpl" class="day69.spring.demo.service.impl.UserServiceImpl"/>
```

```java
// 使用
@Test // 以往控制对象产生
public void testOne() {
  UserService service = new UserServiceImpl();
  service.register();
}

@Test // Spring 接管对象产生
public void testTwo() {
  String configLocation = "applicationContext.xml";
  ClassPathXmlApplicationContext context =
    new ClassPathXmlApplicationContext(configLocation);
  
  // 不需要自己 new 对象了，只要从容器中拿出来
  UserService service = (UserService) context.getBean("userServiceImpl");
  service.register();
}
```

#### DI *dependicy injection*

依赖注入，给 Bean 成员变量赋值，解耦

搭建 DI 项目

* 导包（4 + 1）
  * 同上
* 新建类
* 配置文件
  * `src/applicationContext.xml`

```java
// 手动注入，为 Bean 添加 set 方法
@Test // 手动注入
public void testOne() {
  ClassPathXmlApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");
  UserServiceImpl userService = (UserServiceImpl) context.getBean("userServiceImpl");
  UserDao dao = (UserDao) context.getBean("userDaoImpl");
  userService.setDao(dao);
  userService.register();
}
```

依赖注入

* **通过 `property` `name` 拼接 `set` 方法！！！**

```xml
<!-- applicationContext.xml -->
<!-- 装配 bean -->
<bean id="userServiceImpl" class="day69.spring.demo.service.impl.UserServiceImpl">
	<property name="dao" ref="userDaoImpl"/>
</bean>
<bean id="userDaoImpl" class="day69.spring.demo.dao.impl.UserDaoImpl"/>
```

```java
// 使用
@Test // Spring 依赖注入
public void testTwo() {
  ClassPathXmlApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");
  UserService service = (UserService) context.getBean("userServiceImpl");
  service.register();
}
```

 #### ApplicationContext

接口 **ApplicationContext**

* `ClassPathXmlApplicationContext`  `src` 目录下查找配置文件
* `FileSystemXmlApplicationContext`  项目根目录（`wordking dir`）目录下查找配置文件
* `AnnotationConfigApplicationContext(String basePackage)`  注解配置 **Spring** 不需要 **xml** 配置文件

#### 基于 XML 的装配 Bean（依赖注入）

**Bean** 的实例化方式

* 默认构造

  * **Spring** 使用默认构造方法来实例化 **Bean**

  ```xml
  <bean id="userServiceImpl" class="day69.spring.demo.service.impl.UserServiceImpl">
  	<property name="dao" ref="userDaoImpl"/>
  </bean>
  <bean id="userDaoImpl" class="day69.spring.demo.dao.impl.UserDaoImpl"/>
  ```

* 静态工厂

  * **Spring** 整合其他框架或工具类
  * 方便用 **Spring** 重构以往代码
  * 静态工厂，调用静态方法实例化 **Bean**

  ```xml
  <bean id="userServiceImpl" 
        class="day69.spring.demo.factory.StaticServiceFactory" // 工厂全类名
        factory-method="getUserServiceInstance"> // 工厂静态方法
      <property name="dao" ref="userDaoImpl"/>
  </bean>
  <bean id="userDaoImpl" class="day69.spring.demo.dao.impl.UserDaoImpl"/>
  ```

* 实例工厂

  * 先实例 工厂，再通过工厂非静态方法来实例 **Bean**

  ```xml
  <!-- 先实例工厂 -->
  <bean id="instanceServiceFactory" class="day69.spring.demo.factory.InstanceServiceFactory"/>

  <bean id="userServiceImpl"
        factory-bean="instanceServiceFactory"
        factory-method="getUserServiceInstance">
      <property name="dao" ref="userDaoImpl"/>
  </bean>

  <bean id="userDaoImpl" class="day69.spring.demo.dao.impl.UserDaoImpl"/>
  ```

* `factory-bean` 

  * **Spring**  要求工厂必须实现 `FactoryBean` 接口，实现 `getObject()` 方法生产 **Bean**

#### Bean 的作用域

`<bean id="id" class="class" scope=""/>`

* `singleton` 容器中仅存在一个 **Bean** 实例，**scope 默认值为单例**
  * 加载 **xml** 配置文件就会实例化，`getBean()` 拿出来用
* `prototype` 每次从容器中返回 **Bean** 时，返回一个新的实例
  * 加载 **xml** 配置文件不会实例化，`getBean()` 实例一个对象
* `request` 每次 HTTP 请求创建一个新的 **Bean**，作用于 **WebApplicationContext**
* `session` 同一个 HTTP **Session** 共享一个 **Bean**，作用于 **WebApplicationContext**
* `globalSession` 用于 **Portlet** 应用环境，作用于 **WebApplicationContext**

#### Bean 的生命周期

加载 **xml** 配置文件就会实例化 **Singleton Bean** ，`getBean()` 只是拿出来用

对于 protorype 的 bean，每次 `getBean()` 的时候实例化

> Spring 只帮我们管理单例模式 Bean 的**完整**生命周期
>
> 对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期

`Instantiate` => `Populate properties` 

=> `BeanNameAware setBeanName` => `BeanFactoryAware setBeanFactory`

* => `Pre-initialization BeanPostProcessors`  **BeanPostProcessor**  postProcessBeforeInitialization()
  * => `InitializingBean afterPropertiesSet`  **InitializingBean**
    * => `Call custom init-method` **init()**  
* => `Post-initialization BeanPostProcessors` **BeanPostProcessor** postProcessAfterInitialization()
* **=> Bean is ready to use**  `getBean()`
* **=> Container is down**
  * => `DisposableBean destroy()` **DisposableBean**
    * => `Call custom destroy-method` **destroy()**

![lifecycle](https://ws3.sinaimg.cn/large/006tNc79gy1fpjsamy6uoj30nt0cqq4i.jpg)

####BeanPostProcessor 后处理 Bean

实现接口 `BeanPostProcessor` ，对应生命周期，可以返回增强后的 Bean

* `Object postProcessBeforeInitialization(Object bean, String beanName)`
* `Object postProcessAfterInitialization(Object bean, String beanName)`

```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("before...");
        System.out.println("beanName = " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("after...");
        System.out.println("beanName = " + beanName);
        return bean;
    }
}
```

```xml
<!-- 不用写 id -->
<bean class="day69.spring.demo.processor.MyBeanPostProcessor"/>
```

可以单独给 **Bean** 配置 `init()` 和 `destroy()` 方法，对应生命周期

```java 
public void init() {
    System.out.println("UserServiceImpl.init");
}
public void destroy() {
    System.out.println("UserServiceImpl.destroy");
}
```

```xml
<bean id="userServiceImpl" 
      class="day69.spring.demo.service.impl.UserServiceImpl"
      init-method="init" 
      destroy-method="destroy">
    <property name="dao" ref="userDaoImpl"/>
</bean>
```

#### 属性的依赖注入

自动装配

* 构造装配（构造方法注入）
  * 通过 `constructor-arg`  `name` 匹配构造方法中实际参数名和参数类型 注入
* **Setter** 装配
  * 通过 `property` `name` 拼接 `set` 调用 **setter** 方法注入
* `String + 基本类型`  **value**
* `引用类型`  **ref**  id

```java
// bean 
public class User {
    private String name;
    private int age;
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public int getAge() {return age;}
    public void setAge(int age) {this.age = age;}
}
```

```xml
<!-- 构造装配 -->
<!-- 基本类型和 String 用 value -->
<!-- 引用类型用 ref id -->
 <bean id="user" class="day69.spring.demo.bean.User">
     <constructor-arg name="name" value="Tony"/>
     <constructor-arg name="age" value="23"/>
</bean>
```

```xml
<!-- Setter 装配 -->
<!-- 基本类型和 String 用 value -->
<!-- 引用类型用 ref id -->
 <bean id="user" class="day69.spring.demo.bean.User">
     <property name="name" value="Tony"/>
     <property name="age" value="22"/>
</bean>
```

#### 集合注入

**集合无法通过注解注入**

```java
public class CollectionData {
    String[] arrayData;
    List<String> listData;
    Set<String> setData;
    Map<String, String> mapData;
    Properties properties; // 配置文件也能注入
    
    // getter ... 
    // 必须要 setter ... 
}
```

```xml
<bean id="collection" class="day69.spring.demo.bean.CollectionData">
    <property name="arrayData">
        <array>
            <value>array one</value>
            <value>array two</value>
            <value>array three</value>
        </array>
    </property>
    <property name="listData">
        <list>
            <value>list one</value>
            <value>list two</value>
            <value>list three</value>
        </list>
    </property>
    <property name="setData">
        <set>
            <value>set 111</value>
            <value>set 222</value>
            <value>set 333</value>
        </set>
    </property>
    <property name="mapData">
        <map>
            <entry key="map01" value="map one"/>
            <entry key="map02" value="map two"/>
            <entry key="map03" value="map three"/>
        </map>
    </property>
    <property name="properties">
        <props>
            <prop key="driver">jdbc</prop>
            <prop key="url">root</prop>
        </props>
    </property>
</bean>
```

#### 注解配置 Spring

导包(4 + 1 + 1)

* (4 + 1) 见上
* `spring-aop-5.0.4.RELEASE.jar`

引入约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

配置文件

```xml
<context:component-scan base-package="day69.spring.annotation"/>
```

#### 注解 API

**CLASS** `@Component`  ==>  `<bean class="class"/>` 如果是单例使用 `@Autowired` 可以省略 **value**

**CLASS** `@Component("id")`  ==>  `<bean id="id" class="class"/>`

**CLASS** `@Scope("prototype")`  ==>  `<bean scope="prototype"/>`

依赖注入，无须调用 **setter**，可以在成员方法上，也可以在 **Setter** 方法上

* **FIELD** `@Value("hello")`  ==>  `<property name="field" value="hello"/>`  
* **FIELD** `@Resource(name="referId")`  ==>  `<property name="field" ref="referId"/>` 
* **FIELD** `@AutoWired`  自动类型匹配
* **FIELD** `@AutoWired @Qualifier` 指定修饰符，即成员变量名，不能单独使用

生命周期预处理后处理

* **METHOD** `@PostConstruct`  ==> `<bean init-method="init"/>`
* **METHOD** `@PreDestroy`  ==> `<bean destroy-method="destroy"/>`

Web 开发，是 `@Component` 的扩展，可以不用声明 **value** 值

* **CLASS** `@Controller`  控制器
* **CLASS** `@Service`  业务层
* **CLASS**  `@Repository`  数据访问层

```java
// bean
@Scope("prototype")
@Component("user")
public class User {
    @Value("Tony")
    private String username;
    @Value("password")
    private String password;
    @Value("23")
    private int age;
    
    @Autowired // 引用类型，自动匹配，如果是单例，还可以省略 Component value
    private Vip vip;
    
/*  @Resource(name = "vip")
    private Vip vip;        */
    
    // 不用声明 setter
    @PostConstruct
    private void init() {
        System.out.println("User.init");
    }
    @PreDestroy
    private void destroy() {
        System.out.println("User.destroy");
    }
}
```

```java
// 三层架构中使用 Spring 
@Controller("controller")
@WebServlet(name = "Controller", value = "/demo")
public class ControllerServlet extends HttpServlet {
    @Autowired
    UserService service;
    // ...
}

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao dao;
    // ...
}

@Repository
public class UserDaoImpl implements UserDao {
    // ...
}

 @Test
public void test() {
    ApplicationContext context = new 
        ClassPathXmlApplicationContext("annotationDemo.xml");
    ControllerServlet controller = (ControllerServlet) context.getBean("controller");
    controller.doService();
}
```
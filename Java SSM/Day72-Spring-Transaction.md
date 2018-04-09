# Spring Transaction

#### 内容摘要

Transaction，Spring Transaction，TransactionStatus，TransactionDefinition，PlatformTransactionManeger

导包，第一种，第二种，第三种，**第四种：注解 @Transactional**

------

#### Transaction 回顾

> #### 事务的特性 ACID
>
> * **原子性** **Atomic**
>   * 一系列操作要么成功，要么失败
> * **一致性** **Consistency**
>   * 从一个一致性变换到另一个一致性状态
> * **隔离性** **Isolation**
>   * 多个用户并发访问数据库时，为每一个用户开启事务，事务隔离
>   * 事务之间不能相互干扰
> * **持久性** **Durability**
>   * 一个事务一旦提交，其在数据库中的磁盘是永久性的
>   * 可以在任何地点任何时间，数据的备份与恢复
>
> #### 事务的隔离性 Isolation 
>
> **事务是并发控制的基本单位**
>
> * 隔离低的危险
>   * **脏读**：一个事务读取了另一个事务未提交的数据
>   * **不可重复读**：一个事务读取到另一个事务已经提交的数据
>   * **虚读**：一个事务读取到另一个事务插入的数据
>     * A 事务，查询途中，B 事务，插入操作
> * 隔离级别
>   * `Serializable` : 避免 脏读，不可重复读，虚读 发生，序列化隔离，效率低
>   * **`Repeatable Read` ：避免 脏读，不可重复读 发生，无法避免 虚读，MySQL 默认**
>   * `Read Committed` ：读到已提交数据，避免 脏读，无法避免 不可重复读，虚读
>   * `Read Uncommitted` ：最低级别，读到未提交数据，无法避免 脏读，不可重复读，虚读
> * 设置隔离级别
>   * `set session transaction isolation level [LEVEL]`
> * 查询当前事务隔离级别
>   * `select @@tx_isolation`

#### Spring Transaction

原理还是使用切面编程 AOP

`PlatformTransactionManager`  接口

`TransactionDefinition`

`TransactionStatus`

#### TransactionStatus

获取事务的状态

* `isNewTransaction()` 是否是新事务
* `hasSavePoint()` 是否有回滚点
* `setRollbackOnly()` 设置回滚
* `isRollbackOnly()` 是否回滚
* `flush()` 刷新
* `isCompleted()` 是否完成

#### TransactionDefinition

定义事务的名称，超时时间，只读属性

* 隔离级别
  * `ISOLATION_DEFAULT` 数据库默认隔离级别
  * 其他四个级别见上
* 传播行为
  * 两个业务之间共享事务
  * 开启事务的方法内调用另一个需要事务的方法，嵌套调用时，如何处理两个事务
  * `PROPAGATION_REQUIRED`  支持现有事务，如果没有则新建一个
  * `PROPAGATION_REQUIRED_NEW`  总是发起一个新的事务，如果已存在一个事务，将其挂起
  * 其他级别，查看 API

####PlatformTransactionManager

`DataSourceTransactionManager` **JDBC** 事务管理器，使用 **JdbcTemplate**

`HibernateTransactionManager` **Hibernate** 事务管理器，使用 **Hibernate**

* `TransactionStatus getTransaction(TransactionDefinition difinition)`
  * 根据 事务详情，返回当前事务的状态，管理事务
* `void commit(TransactionStatus status)` 提交
* `void rollback(TransactionStatus status)` 回滚

#### 导包

* 4 + 1 + 1 ： core 、expression、 context、bean、 aop、logging
* DataSource ：jdbc 、C3P0（DBCP）
* JdbcTemplate ：spring-jdbc
* `spring-tx-5.0.4.RELEASE.jar`

#### 第一种 TransactionTemplate

配置

同一个连接 =>  同一个连接池

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```

使用

**DataSourceTransactionManager** 同一个连接池 **DataSource**

```java
@Autowired
private TransactionTemplate transactionTemplate;

@Override
public void transfer(String from, String to, int money) {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult
            (TransactionStatus transactionStatus) {
            doTransfer(from, to, money);
        }
    });
}
```

原理，包裹 **try-catch**，没有异常提交，有异常回滚

```java
 public <T> T execute(TransactionCallback<T> action) throws TransactionException {
     // ...
     TransactionStatus status = this.transactionManager.getTransaction(this);
     Object result;
     try {
         result = action.doInTransaction(status);
     } catch (Error | RuntimeException var5) {
         this.rollbackOnException(status, var5);
         throw var5;
     } catch (Throwable var6) {
         this.rollbackOnException(status, var6);
         throw new UndeclaredThrowableException();
     }
     this.transactionManager.commit(status);
     return result;
     }
 }
```

与 **JDBCTemplate** 使用 **ThreadLocal** 保证同一条连接

```java
// DataSource : ConnectionHolder
// TransactionSynchronizationManager.class ThreadLocal
ThreadLocal<Map<Object, Object>> resources =  // map: dataSource -> ConnectionHolder
    new NamedThreadLocal("Transactional resources");

// DataSourceTransactionManager.class 存放
void doBegin(Object transaction, TransactionDefinition definition) {
	TransactionSynchronizationManager.bindResource(this.obtainDataSource(), 		
                                                   txObject.getConnectionHolder());
}

// JdbcTemplate.class 拿出
Connection con = DataSourceUtils.getConnection(this.obtainDataSource());
// DataSourceUtils.class 拿出
ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
```

#### 第二种 TransactionProxyFactoryBean + Spring AOP

配置 **Service Spring AOP** 代理，增强 **transfer** 方法  `TransactionProxyFactoryBean`

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="proxyService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="proxyInterfaces" value="day72.spring.demo.service.AccountService"/>
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <!-- 要增强的方法  传播行为,隔离级别,只读 READ_ONLY -->
            <prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_REPEATABLE_READ</prop>
        </props>
    </property>
</bean>
```

直接拿到增强后的 **proxyService** 使用

#### 第三种 tx + Spring AspectJ

导包：aopallience、aspect-weaver

引入约束

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" 
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd 
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd 
        http://www.springframework.org/schema/tx 
        http://www.springframework.org/schema/tx/spring-tx.xsd">
```

配置

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 通知 -->
<tx:advice id="transaction" transaction-manager="transactionManager">
    <tx:attributes>
        <!-- 要增强的方法 默认：propagation="REQUIRED" isolation="DEFAULT" -->
        <tx:method name="transfer"/>
    </tx:attributes>
</tx:advice>

<!-- 增强 -->
<aop:config>
    <aop:advisor advice-ref="transaction" 
                 pointcut="execution(* *.*..service.*.transfer(..))"/>
    			<!-- 匹配 service 包下所有类的 transfer 方法 -->
</aop:config>
```

配好就能使用

#### 第四种 tx 注解 @Transactional

配置

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 扫描事务的注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

使用

```java
@Transactional
@Override
public void transfer(String from, String to, int money) {
    doTransfer(from, to, money);
}

// 默认配置
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    boolean readOnly() default false;
    // ...
}
```

检查配置是否生效

`@Transactional(readOnly = true)`  如果抛异常，就可以看到






















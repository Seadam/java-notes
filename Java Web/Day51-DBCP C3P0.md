# DBCP/C3P0

#### 内容摘要

连接池概述，实现连接池，包装设计模式，适配器设计模式，Java EE 连接池规范 DataSource，DBCP，C3P0，配置文件，包装设计模式

---

#### 连接池概述

* 连接池 
  * Connection 创建和销毁需要开销，耗费时间
  * 连接池负责分配、管理、释放数据库连接，重复使用一个现有的连接

#### 实现连接池

* 容器存放连接，链表

```java
// 容器存放连接
private static LinkedList<Connection> connnectionPool;
```

* 获取连接

```java
// 获取连接
public static Connection getConnectionFromPool() {
  if (!connnectionPool.isEmpty()) {
    return connnectionPool.removeFirst();
  } else {
    return null;
  }
}
```

* 回收连接

```java
// 回收连接
public static void returnConnectionToPool(Connection connection) {
  connnectionPool.addLast(connection);
}
```

#### Java EE 连接池规范

* 接口 `javax.sql.DataSource`

* 连接池 (**connection pool**)  ==  数据源 (**data source**)

* 只提供的 **getConnection()** 方法，没有提供放回的方法，建议是调用 **connection.close()** 方法放回

* **包装设计模式**

  *  实现一个 **Connection** 接口的类 **MyConnection**，覆盖 **close()**

  ```java
  public class MyConnection implements Connection {

    	// 拿到当前连接的引用，除 close 方法，均由 connection 调用 
      private Connection connection;
    	// 拿到连接池的引用，调用 close 方法时，将 connection 回收
    	private LinkedList<Connection> pool;

      public MyConnection(Connection connection) {
          this.connection = connection;
      }
    
    	// 重写该方法，回收 connection
    	@Override
      public void close() throws SQLException {
          pool.addLast(connection);
      }

      @Override
      public Statement createStatement() throws SQLException {
          return connection.createStatement();
      }

      @Override
      public PreparedStatement prepareStatement(String sql) throws SQLException {
          return connection.prepareStatement(sql);
      }
    
  	// ... 省略
  }
  ```

* **适配器设计模式**

  * 由于以上包装设计模式，只要覆盖 close() 方法，却要写很多无关代码，这个工作交给适配器来做

  ```java
  // 适配器
  public class ConnectionAdapter implements Connection {

      protected Connection connection;

      public ConnectionAdapter(Connection connection) {
          this.connection = connection;
      }

      @Override
      public Statement createStatement() throws SQLException {
          return connection.createStatement();
      }

      @Override
      public PreparedStatement prepareStatement(String sql) throws SQLException {
          return connection.prepareStatement(sql);
      }
    
   	@Override
      public void close() throws SQLException {
          connection.close();
      }

  	// ... 省略
  }

  // 包装类，只需要继承适配器，覆盖方法即可，让包装类清晰明了
  class MyConnection extends ConnectionAdapter {

      private LinkedList<Connection> pool;

      public MyConnection(Connection connection, LinkedList<Connection> pool) {
          super(connection);
          this.pool = pool;
      }

      @Override
      public void close() throws SQLException {
          pool.addLast(super.connection);
      }
  }
    
  ```

#### DBCP

**Apache** **Database Connection Pool**

* `commons-dbcp.jar`
* `commons-pool.jar`
* 使用 `.properties` 配置文件，并放在 **src** 目录下，需要手动加载

```java
// util
public class DBCPUtil {

    private static DataSource dataSource;

   	// 初始化 DBCP 数据源
    static {
        Properties properties = new Properties();
        InputStream stream = DBCPUtil.class.getClassLoader().getResourceAsStream("dbcp.properties");
        try {
            properties.load(stream);
            dataSource = BasicDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

  	// 通过该方法拿到数据源中连接
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

```properties
# dbcp.properties
# 连接设置
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/jdbc
username=root
password=root123456

# 初始化连接数量
initialSize=10

# 最大连接数量
maxActive=50

# 最大空闲连接
maxIdle=20

# 最小空闲连接
minIdle=5

# 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒
maxWait=60000

# JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;]
# 注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。
# useSSL=false 可以去除不安全连接警告
connectionProperties=useSSL=false

# 指定由连接池所创建的连接的自动提交（auto-commit）状态。
defaultAutoCommit=true

# driver default 指定由连接池所创建的连接的只读（read-only）状态。
# 如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix）
defaultReadOnly=

# driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
# 可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED,  REPEATABLE_READ, SERIALIZABLE
defaultTransactionIsolation=REPEATABLE_READ
```

#### C3P0

* `c3p0-0.9.1.2-jdk1.3.jar`
* 使用 **命名要求** `c3p0-config.xml`  配置文件，并放在 **src** 目录下自动加载

```java
public class C3P0Util {

    private static DataSource dataSource;
    
    static {
      	// 配置文件自动加载，只需要指定加载的配置名
        dataSource = new ComboPooledDataSource("mysql");
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
  <!-- 默认驱动配置 -->
  <default-config>
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/</property>
    <property name="user">root</property>
    <property name="password">123456</property>
    <property name="acquireIncrement">5</property>
    <property name="initialPoolSize">10</property>
    <property name="minPoolSize">5</property>
    <property name="maxPoolSize">20</property>
  </default-config>

  <!-- 定义指定驱动配置名 -->
  <named-config name="mysql">
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/jdbc?useSSL=false</property>
    <property name="user">root</property>
    <property name="password">root123456</property>
    <property name="acquireIncrement">5</property>
    <property name="initialPoolSize">10</property>
    <property name="minPoolSize">5</property>
    <property name="maxPoolSize">20</property>
  </named-config>

  <!-- 其他驱动配置 -->
  <!-- 
    <named-config name="oracle">
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/</property>
        <property name="user">root</property>
        <property name="password">root</property>
        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">10</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">20</property>
    </named-config>
    -->
</c3p0-config>
```

#### DBCP ／ C3P0 配置文件问题

* 注意事项
  * 对于不同的连接池框架，由于配置命名不同，在代码中会导致 方法名不同，要注意
  * **get / set**  + 配置名


* 命名区别

  | 命名           | DBCP              | C3P0          |
  | ------------ | ----------------- | ------------- |
  | **driver**   | *driverClassName* | *driverClass* |
  | **url**      | *url*             | *jdbcUrl*     |
  | **username** | *username*        | *user*        |

#### 包装设计模式

在不修改原有类代码的情况下，为该类增加新功能

1. 持有该类引用，并实现该类接口，具体功能由该类实现，新功能覆盖方法即可
2. 继承该类，重写父类方法
















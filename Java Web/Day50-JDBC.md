# JDBC

#### 内容摘要

补充，数据库访问过程，JDBC 概述，JDBC 使用过程，JUnit4，JDBC 使用过程，注册驱动，建立连接，发送请求，解析结果集，释放资源，SQL 注入，批处理，MySQL 事务，事务的特性 **ACID**，事务的隔离性，try-with-resource 语句

---

#### 补充

* 注释 

  ```mysql
  -- 单行注释
  /* 
  	多行注释
  */
  ```

* 图形界面 SQL yony

* 关键字 **LIMIT** 

  * 结果表限制显示行数  
  * **`limit m,n`  从 m + 1 行开始的 n 条数据**  **(m,m + n]**
  * **主要用在分页加载**

  ```mysql
  -- 显示 2 条
  select * from t_name LIMIT 2;
  -- 显示第三行开始 10 条数据
  select * from t_name LIMIT 2, 10;
  ```

#### 数据库访问过程

1. 建立与数据库服务器的连接
2. 构建 SQL 语句，向服务器发送 SQL 语句，操作数据库
3. 服务器处理客户端请求，向客户端返回处理结果
4. 客户端解析收到的结果集，并作相应处理
5. 释放相关资源

#### 概述

* **JDBC：Java Database Connectivity**
* Java 操作数据库的规范
  * 各种数据库服务端实现不同，**JDBC** 为上层提供统一实现接口
  * **数据库驱动**：各个服务器厂商对 **JDBC** 规范的实现
* `java.sql`
* `javax.sql`

#### JUnit 4

* 单元测试工具
* 导包 `hamcrest-core-1.3.jar` `junit-4.12.jar`  **IDEA** 会自动下载
* 使用
  * **IDEA**  `Shift + Comman + t` 创建 测试类
  * 方法前面加注解  **@Test**
* 其他 API 
  * **TestCase**

```java
import org.junit.Test;

public class JUnit4Test {

    @Test
    public void helloJUnit4() {
        System.out.println("Hello JUnit4.");
    }
}
```

#### JDBC 使用过程

1. 导入数据库驱动 `xxxx.jar`  由`DriverManager` 注册驱动，建立连接

2. 注册驱动 **Driver**

   * `com.mysql.jdbc.Driver`

   ```Java
   DriverManager.registerDriver(new Driver());
   ```

3. 建立连接 **Connection**

   * **url**  `jdbc:mysql://localhost:3306/jdbc`
     * jdbc 主协议
     * mysql 子协议
     * localhost:3306 主机及端口
     * jdbc 要连接的数据库名
     * `[?参数名=参数值]`
   * **user**
   * **password**

   ```java
   String url = "jdbc:mysql://localhost:3306/jdbc";
   String user = "root";
   String password = "root123456";
   Connection connection = DriverManager.getConnection(url, user, password);
   ```

4. 发送请求 **Statement**

   ```java
   // 构造 SQL 语句
   String sql = "insert into t_user values(null, 'Tony', 18);";   

   // 发送语句
   Statement statement = connection.createStatement();
   statement.executeUpdate(sql);
   ```

5. 解析结果集（针对查询语句）

   * 见下 **ResultSet**

6. 释放资源

   ```Java
   statement.close();
   connection.close();
   ```

#### 注册驱动 Driver

* `com.mysql.jdbc.Driver`
* 可以放配置文件，方便快捷替换，见下建立连接的第三种方式
* **Driver** 类会在静态代码块自己注册自己，所以只需要触发他的类加载器就可以达到注册目的
* **Class.forName("com.mysql.jdbc.Driver");**

```java
// Driver() 源码
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```

```java
 static {
   // 1. 注册驱动
   // DriverManager.registerDriver(new Driver());
   try {
     // 触发类加载，仅执行 Driver 静态代码块，注册驱动
     Class.forName("com.mysql.jdbc.Driver");
   } catch (ClassNotFoundException e) {
     System.out.println();
     e.printStackTrace();
   }
 }
```

#### 建立连接 Connection

`java.sql.Connection`  为了无缝切换不同数据库产品

> WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

* 解决方法：  `String url = "jdbc:mysql://localhost:3306?useSSL=false";`
  * **?useSSL=false**  参数


* **url** 第二种写法，如果 **ip** 和 **port** 为 `localhost:3306` 时，可以省略

```java
String url = "jdbc:mysql:///jdbc";
// url=jdbc:mysql://localhost:3306/jdbc?useSSL=false
```

* 第二种方式

```java
// 简洁写法，传参数
// ?user=root&password=root123456
String url = "jdbc:mysql://localhost:3306/jdbc?user=root&password=root123456";
Connection connection = DriverManager.getConnection(url);
```

* 第三种方式

```java 
// 使用配置文件，放在 src 目录下， classloader 读入
 private static final Properties properties = new Properties();

static {
  // 注册驱动
  // DriverManager.registerDriver(new Driver());
  // 使用配置文件读取驱动名称，可以灵活更改驱动
  InputStream stream = JDBCDemo.class.getClassLoader().getResourceAsStream("jdbc.properties");
  try {
    properties.load(stream);
    Class.forName(properties.getProperty("driver"));
  } catch (ClassNotFoundException | IOException e) {
    e.printStackTrace();
  }
}

private Connection getConnection() throws SQLException {
  // 使用配置文件连接数据库，灵活
  return DriverManager.getConnection(properties.getProperty("url"), properties);
}
```

#### 发送请求 Statement

`java.sql.Statement`

`Statement statement = connection.createStatement()`

* `int executeUpdate(String sql)`
  * **增删改操作**
  * 返回成功行数
* `ResultSet executeQuery(String sql)`
  * **查询操作**
  * 返回结果集 **ResultSet**

```java
// 插入
String insert = "";
int status = statement.executeUpdate(insert);
// 查询
String select = "";
ResultSet resultSet = statement.executeQuery(select);
```

#### ReslutSet

* 封装 **JDBC** 查询结果集，采用类似于表格的格式，内部维护一个**游标**，默认指向表格第一行之前
* `boolean next()`
  * 游标移动到下一行，如果有数据返回 true，否则返回 flase
* `Object get(String columnLabel)`
  * 通过某一列**列名** **columnLabel** 返回当前行的数据
* `Object get(int columnIndex)`
  * 通过**列编号** **columnIndex** 返回当前行的数据，列编号从左往右编号，编号从 **1** 开始
* 游标移动 API
  * `previous()`
  * `beforeFirst()`
    * 到第一行之前
  * `afterLast()`
    * 到最后一行之后，用于逆序遍历
  * `absolute(int row)`
    * 跳转 行编号
    * 指标指向第 **0** 行，行编号从 **1** 开始

#### 解析结果集

```java 
// 1. 通过列名某一列 当前行的数据
while (resultSet.next()) {
  int id = resultSet.getInt("id");
  String name = resultSet.getString("name");
  int age = resultSet.getInt("age");
}

// 2. 通过列编号获得 当前行的数据，属性列从左往右编号，编号从 1 开始
while(resultSet.next()) { 
  int id = resultSet.getInt(1);
  String name = resultSet.getString(2);
  int age = resultSet.getInt(3);
}
```

#### 释放资源

放在 **finally** 语句中释放

* **Connection**
  * **晚创建、早释放**
* **Statement**
* **ResultSet**

```java
finally {
  connection.close();
  statement.close();
  resultSet.close();
}
public static void release(Connection conn, Statement st, ResultSet rs) {
  closeQuietly(conn);
  closeQuietly(st);
  closeQuietly(rs);
}

private static void closeQuietly(AutoCloseable ac) {
  if (null != ac) {
    try {
      ac.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

#### SQL 注入

* 现象： SQL 语句的参数与 SQL 命令拼接成一条 SQL 语句进行发送给服务器
  * **用户输入也被当作 SQL 语句来执行**
* 解决方法：禁止 SQL 参数当作命令执行，**参数与命令分离**
* 使用 **PrepareStatement**
* `prepareStatement(String querySql)`
  * 客户端与数据库服务端会交互两次
  * 第一次发送纯 SQL 命令，编译和优化 SQL 命令，**并缓存**编译和优化结果
  * 第二次发送纯 SQL 参数，拼接参数，执行 SQL 语句
* **querySql** 占位符 `?`
  * 将占位符从左到右编号，编号从 **1** 开始
* `setString(int parameterIndex, String value)`
  * **parameterIndex**  即占位符参数，编号从 **1** 开始

```java
// SQL 注入，会让判断密码注释掉，并且 1 = 1 为真
String username = "root' OR 1 = 1 -- ";
```

```java 
// 纯 sql 命令
String querySql = "select * from user where username = ? and password = ?";
// 第一次发送纯 sql 命令
PrepareStatement prepareStatement = connection.prepareStatement(querySql);
// 第二次发送纯 sql 参数
prepareStatement.setString(1, "root");
prepareStatement.setString(2, "root");
// 发送
prepareStatement.executeQuery();
```

#### 批处理 Batch

**针对增删改**

* `statement.addBatch(String sql)`
  * 不同 SQL 命令
  * 灵活性高
* `prepareStatement.addBatch()`
  * 编译一次 SQL 命令，拼接不同 SQL 参数，效率高
  * 针对同样的 SQL 命令，不同的 SQL 参数
* `int[] executeBatch()`
  * 返回每条语句成功更改行数
* `clearBatch()`

#### Statement / PrepareStatement 比较

* **Statement** 
  * 单次执行，效率高
  * 不安全，SQL 注入问题
* **PrepareStatement**
  * 批处理，同一条 SQL 命令，效率高
  * 安全，防止 SQL 注入

#### try - with - resource 语句

* `try - with - resource ( )`  只能使用 AutoCloseable 接口对象，如果要传参，可以调用方法

```java
// 方法返回  AutoCloseable 接口对象
@Test
public void tryWithResourceTest() {
  try (
    Connection connection = DBUtil.getConnection();
    PreparedStatement statement = getPre(connection);
    ResultSet resultSet = statement.executeQuery();
  ) {
    while (resultSet.next()) {
      // 解析结果集
    }
  } catch (SQLException e) {
    e.printStackTrace();
  }
}
private PreparedStatement getPre(Connection connection) throws SQLException {
  String querySql = "SELECT * FROM t_user WHERE `username` = ? AND `password` = ?;";
  PreparedStatement statement = connection.prepareStatement(querySql);
  statement.setString(1, "username"); // 此处 username 通过参数传递过来
  statement.setString(2, "password"); // 此处 password 通过参数传递过来
  return statement;
}
```

#### 事务处理

* 事务

  * 原子性操作，要么全部完成，要么全部不完成
  * **隔离性，多线性安全**

* 开启事务

  ```mysql
  -- 开启事务
  start transaction;

  -- 事务操作，多条 SQL 语句
  update acount set money = money - 100 where name = 'customer';
  update acount set money = money + 100 where name = 'producer';

  -- 事务提交
  commit;

  -- 事务回滚，清除所有即将要提交的事务
  rollback;
  ```

*  **JDBC**

  * `connection.setAutoCommit(false)`   **start transaction **   或    **BEGIN**
  * `connection.rollback()`  **rollback**  在事务还没提交前回到最开始状态或回滚点
  * `connection.commit()` **commit**
  * `SavePoint savePoint = connection.setSavePoint()`
    * 回滚点
    * 事务执行途中，可以保存回滚点，一旦发生异常，可以回滚到回滚点前已经添加的事务

#### 事务的特性 ACID

* **原子性** **Atomic**
  * 一系列操作要么成功，要么失败
* **一致性** **Consistency**
  * 从一个一致性变换到另一个一致性状态
* **隔离性** **Isolation**
  * 多个用户并发访问数据库时，为每一个用户开启事务，事务隔离
  * 事务之间不能相互干扰
* **持久性** **Durability**
  * 一个事务一旦提交，其在数据库中的磁盘是永久性的
  * 可以在任何地点任何时间，数据的备份与恢复

#### 事务的隔离性 Isolation

**事务是并发控制的基本单位**

* 隔离低的危险
  * **脏读**：一个事务读取了另一个事务未提交的数据
  * **不可重复读**：一个事务读取到另一个事务已经提交的数据
  * **虚读**：一个事务读取到另一个事务插入的数据
    * A 事务，查询途中，B 事务，插入操作
* 隔离级别
  * `Serializable` : 避免 脏读，不可重复读，虚读 发生，序列化隔离
  * **`Repeatable Read` ：避免 脏读，不可重复读 发生，无法避免 虚读，MySQL 默认**
  * `Read Committed` ：避免 脏读，无法避免 不可重复读，虚读
  * `Read Uncommitted` ：最低级别，读未提交，无法避免 脏读，不可重复读，虚读
* 设置隔离级别
  * `set session transaction isolation level [LEVEL]`
* 查询当前事务隔离级别
  * `select @@tx_isolation`

#### MySQL 事务

[source](http://blog.csdn.net/javazejian/article/details/69857949)

```mysql
-- 关闭自动提交功能
SET AUTOCOMMIT=0;
-- 开启自动提交功能
SET AUTOCOMMIT=1;
```

* **悲观锁**：假设会发生并发冲突，回避一切可能违反数据完整性的操作
* **乐观锁**：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性，注意乐观锁不能解决脏读问题
  * 乐观锁，大多情况下是基于数据版本（ Version ）记录机制实现


* **共享锁定**：将对象数据变为只读形式的锁定，这样就允许多方同时读取一个数据，此时数据将无法修改
* **排他锁定**：在对数据进行 insert / update / delete 时进行锁定，在此时其他用户(进程或事务)一律不能读取数据，从而也保证数据完整性


# Transaction

#### 内容摘要

DBUtils 事务处理，ThreadLocal，Session 复习，Cookie 复习

---

#### DBUtils 事务处理

* 同一个连接
* `setAutoCommit(false)`

```java
// 转账模拟，在 DAO 层开启事务处理
public boolean transfer(String from, String to, int money) {
  Connection connection = null;
  QueryRunner queryRunner = new QueryRunner();
  boolean flag = false;

  try {
    connection = C3P0Util.getConnection();
    // 开启事务，必须是同一个连接
    connection.setAutoCommit(false);

    // from - money
    String sql = "update account set money = money - ? where name = ?";
    queryRunner.update(connection, sql, money, from);

    // 可能发生异常 int i = 1/0;

    // to - money
    sql = "update account set money = money + ? where name = ?";
    queryRunner.update(connection, sql, money, to);

    // 提交事务
    DbUtils.commitAndCloseQuietly(connection);
    flag = true;
  } catch (Exception e) {
    e.printStackTrace();
    // 事务中出现异常，回滚
    if (null != connection) {
      DbUtils.rollbackAndCloseQuietly(connection);
    }
  }
  return flag;
}

// Dao 层专注于与数据库交互
public Account getAccount(String name) throws SQLException {
  QueryRunner queryRunner = new QueryRunner();
  String sql = "select * from account where name = ?";
  return queryRunner.query(TransactionUtil.getConnection(),
                           sql, new BeanHandler<>(Account.class), name);
}

@Override
public void updateAccount(Account account) throws SQLException {
  QueryRunner queryRunner = new QueryRunner();
  String sql = "update account set money = ? where name = ?";
  queryRunner.update(TransactionUtil.getConnection(),
                     sql, account.getMoney(), account.getName());
}
```

* 对于分层来说，事务应该在 **Service** 层处理

```java
// 模拟转账，在 Service 层开启事务
public boolean transfer(String from, String to, int money) {
  boolean flag = false;
  Connection connection = null;

  try {
    connection = C3P0Util.getConnection();

    // 开启事务
    connection.setAutoCommit(false);
    TransactionUtil.setConnection(connection);

    // 检查账户余额
    Account fromAccount = dao.getAccount(from);
    Account toAccount = dao.getAccount(to);

    // 转账业务 from - money to + money
    fromAccount.setMoney(fromAccount.getMoney() - money);
    toAccount.setMoney(toAccount.getMoney() + money);

    // 提交事务，写到数据库
    dao.updateAccount(fromAccount);
    // 此处发生事务异常 int i = 1/0;
    dao.updateAccount(toAccount);

    // 提交事务
    DbUtils.commitAndCloseQuietly(connection);
    flag = true;
  } catch (Exception e) {
    e.printStackTrace();
    // 事务中出现异常，回滚
    if (null != connection) {
      DbUtils.rollbackAndCloseQuietly(connection);
    }
  }

  return flag;
}
```

#### ThreadLocal

`java.lang.ThreadLocal`

解决线程安全问题，线程局部变量

* `get()`
* `set()`

```java
public class TransactionUtil {
    
    private static final ThreadLocal<Connection> threadLoacal = new ThreadLocal<>();
    
    public static void setConnection(Connection connection) {
        threadLoacal.set(connection);
    }
    
    public static Connection getConnection() {
        return threadLoacal.get();
    }
}
```



### Session 复习

问题描述：如何让用户关闭浏览器，再重新打开，**Session** 中保存的信息还存在

[reference](https://www.jianshu.com/p/2879fb0a5b2e)

**Session** 何时被删除

* `request.getSession().invalidate()`
* 距离上一次收到客户端发送的 **JSessionId** 间隔超过了最大空闲有效时间
* 服务器进程被停止

注意：

* **关闭浏览器只会使存储在用户浏览器内存中的 JSessionId Cookie 失效**
* **不会使服务器端的 Session 对象失效**

要点：

* **Session** 在创建时，会给浏览器发送唯一标识当前用户 **Session**  **JSessionId** **Cookie**
* 浏览器请求时会带上 **JSessionId** 告诉浏览器当前会话状态
* 默认情况下该 **JSeesionId Cookie** 在浏览器关闭时销毁
* 当重新打开浏览器时，若要会话信息 **Session** 还存在的话
  * 原因：浏览器一关闭就立即销毁默认 **JSessionId** **Cookie**
  * 解决：将 **JSessionId** 手动发送给用户浏览器，并设置过期时间

```java
// 拿到当前 Session 对象
HttpSession session = request.getSession();
// 拿到 sessionId
String sessionId = session.getId();
// 创建 Cookie
Cookie cookie = new Cookie("JSessionId", sessionId);
// 设置 JSessionID 过期时间，单位 秒
cookie.setMaxAge(60 * 30); // 半小时
// 写回浏览器
response.addCookie(cookie);
```

### Cookie 复习

键值对

* 创建 Cookie
* 删除 Cookie
* 拿到 Cookie

```java
// 创建 Cookie
Cookie usernameCookie = new Cookie("username", username);
response.add(usernameCookie);

// 删除 Cookie
Cookie cookie = new Cookie("username", null);
cookie.setPath(contentPath);
cookie.setMaxAge(0);
response.add(usernameCookie);

// 拿到 Cookie Servlet
Cookie[] cookies = request.getCookies();
for (Cookie cookie : cookies) {
    if ("username".equals(cookie.getName())) {
      	username = cookie.getValue();
    }
}

// 拿到 Cookie Jsp
${cookie.username.value}
```











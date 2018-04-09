# Design Pattern

#### 内容摘要

设计模式介绍，软件设计原则，经典设计模式，单例模式，工厂模式，建造者模式，装饰者模式，适配器，**代理**

---

#### 设计模式介绍

* 软件开发过程中总结的经验
* 被反复使用的、多人知晓的、经过分类编目的、代码设计经验总结
* **解耦**

#### 软件设计原则

* 设计方式：**提出抽象，隔离变化**
* 5 大设计原则，**SOLID**
  * **S** ingle - 单一职责
    * 一个模块负责一个功能
    * 一个类负责一个业务
    * 一个接口实现一个功能
  * **O** pen - 开放封闭原则
    * 对扩展开放，新功能，增加新代码（新的类）
    * 对修改封闭，不建议修改底层 API，牵一发而动全身s
  * **L** iskov - 李氏替换原则
    * 任何一个基类可以出现的地方，子类一定可以出现
    * 父类引用指向子类实例
  * **I** ndependency - 接口隔离原则
    * 不同功能的接口，放在不同的 **interface** 里
  * **D** ependency - 依赖倒置原则（Spring、IOC、ASPECT）
    * 具体依赖于抽象
    * 先设计抽象的 DAO 和 service，再实现接口的实现类

#### 经典设计模式 *Design Pattern*

GOF ，Gang of Four ，四人帮，23 种设计模式

创建型（Creational Patterns）

* **单例模式、创造者模式、工厂方法模式、抽象工厂模式**、原型模式

结构型（Structural Patterns）

* **适配器模式、装饰器模式、代理模式**、外观模式、桥接模式、组合模式、享元模式

行为型（Behavioral Patterns）

* 策略模式、模版方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式



#### 单例模式（Singleton）

特点

* 单例类只能有一个实例
* 单例类必须自己创建自己的唯一实例
* 单例类必须给外界提供这一实例

优点

* 节省资源
* 需要一个唯一对象的时候

缺点

* 隔离原则要求，应该是不考虑外界的使用，而非由单例内部实现

实现

```java
// 线程不安全的懒汉
public class LazySingleton {
    private static LazySingleton singleton;

    private LazySingleton(){}

    public static LazySingleton getSingleton() {
        if (null == singleton) {
            singleton = new LazySingleton();
        }
        return singleton;
    }
}
```
```java
// 线程安全的懒汉
public class SafeLazySingleton {
    private static SafeLazySingleton singleton;

    private SafeLazySingleton(){}

  	// 加锁，效率不高
    public static synchronized SafeLazySingleton getSingleton() {
        if (null == singleton) {
            singleton = new SafeLazySingleton();
        }
        return singleton;
    }
}
```
```java
// 线程安全的饿汉，立即加载
public class StarveSingleton {
    private static StarveSingleton singleton;
	// 也可以 private static StarveSingleton singleton = new StarveSingleton();
    static {
        singleton = new StarveSingleton();
    }

    private StarveSingleton(){}

    public static StarveSingleton getSingleton() {
        return singleton;
    }
}
```
```java
// 线程安全的懒加载，静态内部类，（嵌套类），推荐使用
public class InnerLazySingleton {
  	// 私有化构造方法，防止外部创建对象
    private InnerLazySingleton(){}

  	// 静态内部类，当调用静态方法才会实例化该对象，实现懒加载
    // 静态内部类，线程安全
    private static class Holder {
        static InnerLazySingleton innerLazySingleton = new InnerLazySingleton();
    }
  
  	// 返回内部唯一实例
    public static InnerLazySingleton getSingleton() {
        return Holder.innerLazySingleton;
    }
}
```

#### 工厂模式（Factory）

特点

* 创建对象时，不必知道具体创建逻辑
* 通过使用一个共同的接口创建新对象

简单工厂

* 优点：屏蔽具体创建细节，修改一处就好
* 缺点：当要扩展类时，需要更改已经写好的代码，违反开放封闭原则

```java
// Bean 类
public abstract class Fruit {}
public class Apple extends Fruit {}
public class Orange extends Fruit {}
public class Banana extends Fruit {}

public class SimpleFactory {
    public static Fruit getInstance(String name) {
        switch (name) {
            case "apple":
                return new Apple();
            case "orange":
                return new Orange();
            case "banana":
                return new Banana();
            default:
                return null;
        }
    }
}
```

工厂方法

* 优点：扩展类时，不用修改原有代码，只需要增加新类即可
* 缺点：类变多了，维护问题

```java
public abstract class AbstractFactory {
    public abstract Fruit getInstance();
}

public class AppleFactory extends AbstractFactory {
    @Override
    public Fruit getInstance() {
        return new Apple();
    }
}

public class OrangeFactory extends AbstractFactory {
    @Override
    public Fruit getInstance() {
        return new Orange();
    }
}

public class BananaFactory extends AbstractFactory {
    @Override
    public Fruit getInstance() {
        return new Banana();
    }
}
```

#### 建造者模式（Builder）

将一个复杂对象的构建与其实现分离，同样构建过程可以创建不同的实例

* **Builder** 接口
* **ConcreteBuilder** 实现类
* **Director** 构建对象流程，塞数据
* **Product** 实际要创建的对象

```java
// Builder
public interface Builder<T> {
    T build();
}

// Product
public class User {
    private User () {}
    private String firstName;
    private String lastName;
    private String gender;
    private int age;

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public String getGender() {
        return gender;
    }

    public int getAge() {
        return age;
    }

    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.gender = builder.gender;
        this.age = builder.age;
    }
    
    public static UserBuilder builder() {
        return new UserBuilder();
    }

    // ConcreteBuilder
    static class UserBuilder implements Builder<User> {
        private String firstName;
        private String lastName;
        private String gender;
        private int age;

        public UserBuilder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }

        public UserBuilder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }

        public UserBuilder gender(String gender) {
            this.gender = gender;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        @Override
        public User build() {
            return new User(this);
        }
    }
}

// Director
 User user = User.builder()
                .firstName("Tony")
                .lastName("Shites")
                .age(23)
                .gender("male")
                .build();
```

#### 装饰者模式（Decorator）包装

* 定义一个类，实现与被包装（**增强**）对象所实现的接口
* 定义一个变量，引用被包装（**增强**）对象
* 构造方法接收被包装（**增强**）对象
* 覆盖要被增强的方法
* 对于不用增强的方法，调用被包装（**增强**）对象对应的方法

```java
public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        // 替换请求参数，装饰者设计模式
        // 1. 自定义 request 对象，继承自 HttpServletRequestWrapper
        MyRequest myRequest = new MyRequest(req);
        // 2. 重写 getParameter(String name) 方法
        // 3. 传递新的 request 对象
        chain.doFilter(myRequest, resp);
}

class MyRequest extends HttpServletRequestWrapper {

    public MyRequest(ServletRequest request) {
        super((HttpServletRequest) request);
    }

    @Override
    public String getParameter(String name) {
        if ("message".equals(name)) {
            String s = super.getParameter(name);
            return s.replace("TMD", "挺萌的");
        }
        return super.getParameter(name);
    }
}
```

#### 适配器模式（Adapter）

* 适配

#### 代理模式（Proxy）

**代理类负责为委托类预处理消息，过滤消息并转发消息，以及委托类执行完毕后的后续处理**

* 优点：
  * 降低各个功能模块之间的耦合度，提高开发的效率和方便程序的维护度
  * 减少代码量
  * 不关注目标的具体实现
* **静态代理——包装模式**
* 动态代理——直接给目标类动态生成一个代理对象，而不需要手写代理类
* 实现
  * **JDK Proxy** 类，为实现了某些接口的对象，生成代理对象
  * **cglib （Spring）**，为没有实现接口的对象，生成代理对象

**JDK** 动态代理实现

* `Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`
  * **ClassLoader** 被代理对象的类加载器
  * **interfaces** 被代理对象实现的接口的数组
  * **InvacationHandler** 代理动作执行
    * `Object invoke(Object proxy, Method method, Object[] args)`
    * **proxy** 调用该方法的代理对象
    * **method** 被代理对象的接口的方法，接口中所有方法
    * **args** 调用被代理对象的方法时传入的参数

```java
// 允许在当前包下生成代理类文件，可以看到部分 class 文件
System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```

```java
// 被代理类及接口
public interface HouseKeeper {
    int makeContract();
}
public class MyHouseKeeper implements HouseKeeper {
    @Override
    public int makeContract() {
        System.out.println("Sign a contract...");
        return 0;
    }
}
```
```java
// 静态代理 包装
// 为 MyHouseKeeper 生成代理类
public class StaticProxyHouseKeeper implements HouseKeeper {
    HouseKeeper houseKeeper;
    public StaticProxyHouseKeeper(HouseKeeper houseKeeper) {
        this.houseKeeper = houseKeeper;
    }
    @Override
    public int makeContract() {
        System.out.println("Proxy start...");
        System.out.println("Do something...");
        int i = houseKeeper.makeContract();
        System.out.println("Proxy end...");
        System.out.println("Do something...");
        return i;
    }
}
@Test
public void staticProxy() {
  HouseKeeper houseKeeper = new MyHouseKeeper();
  StaticProxyHouseKeeper proxyHouseKeeper = new StaticProxyHouseKeeper(houseKeeper);
  proxyHouseKeeper.makeContract();
}
```

```java
// 动态代理
// java.lang.reflect.Proxy
@Test
public void dynamicProxy() {
  HouseKeeper houseKeeper = new MyHouseKeeper();
  HouseKeeper proxyHouseKeeper = (HouseKeeper) Proxy.newProxyInstance(
    houseKeeper.getClass().getClassLoader(),
    houseKeeper.getClass().getInterfaces(),
    new InvocationHandler() {
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) 								throws Throwable {
        System.out.println("start proxy...");
        Object invoke = method.invoke(houseKeeper, args);
        System.out.println("end proxy...");
        return invoke;
      }
    }
  );
  proxyHouseKeeper.makeContract();
}
```

```java
// 实现 InvocationHandler 接口写法
public class DynamicProxyHouseKeeper implements InvocationHandler {
    private Object originObject;
    public Object bind(Object originClass) {
        this.originObject = originClass;
        return Proxy.newProxyInstance(
                originClass.getClass().getClassLoader(),
                originClass.getClass().getInterfaces(),
                this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object invoke = method.invoke(originObject, args);
        if (null != invoke) {
            result(invoke);
        }
        after();
        return invoke;
    }
    private void after() {
        System.out.println("proxy end...");
    }
    private void result(Object invoke) {
        System.out.println("result = " + invoke);
    }
    private void before() {
        System.out.println("proxy start...");
    }
}

@Test
public void proxy() {
  HouseKeeper houseKeeper = new MyHouseKeeper();
  HouseKeeper proxy = (HouseKeeper) new DynamicProxyHouseKeeper().bind(houseKeeper);
  proxy.makeContract();
}
```

```java
// proxy
public final class $Proxy1 extends Proxy implements HouseKeeper {
  private InvacationHandler h;
  private $Proxy1() {}
  public $Proxy1(InvacationHandler h) {
    this.h = h;
  }
  public int makeContract() {
    // 创建 Method 对象
    Method method = HouseKeeper.class.getMethod("makeContract", new Class[]{int.class});
	// 调用 InvacationHandler invoke 方法，回调
    return (Integer) h.invoke(this, method, new Object[0]);
  }
}
```

#### 代理模式事务处理

```java
// TransactionUtils
// 开启事务
public static void beginTransaction() throws SQLException {
  getConnection().setAutoCommit(false);
}

// 提交事务
public static void commitTransaction() {
  DbUtils.commitAndCloseQuietly(getConnection());
}

// 回滚事务
public static void rollbackTransaction() {
  DbUtils.rollbackAndCloseQuietly(getConnection());
}

public static Connection getConnection() {
  Connection connection = threadLocal.get();
  if (null == connection) {
    try {
      connection = C3P0Util.getConnection();
    } catch (SQLException e) {
      e.printStackTrace();
    }
    threadLocal.set(connection);
  }
  return connection;
}

// proxy
@Override
public AccountService getTransactionInstance() {
  return (AccountService) Proxy.newProxyInstance(
    service.getClass().getClassLoader(),
    service.getClass().getInterfaces(),
    new InvocationHandler() {
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) {
        boolean flag = false;
        try {
          TransactionUtil.beginTransaction();
          method.invoke(service, args);
          TransactionUtil.commitTransaction();
          flag = true;
        } catch (Exception e) {
          System.out.println("代理捕获到异常：" + e.getCause());
          TransactionUtil.rollbackTransaction();
        }
        return flag;
      }
    }
  );
}
```


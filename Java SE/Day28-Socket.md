# Java Socket

#### 内容摘要

Closeable，AutoCloseable，TCP传输，反射，类加载器，反射，获取字节码文件对象，Class，Constructor

---

#### Closeable   AutoCloseable

* `AutoCloseable`

  * 幂等方法，即调用多次效果一样

* `Closeable`

  * 非幂等方法，只能调用一次，多次调用可能会影响调用结果

* `try - with - resource`

  * 实现了 `Closeable` 或 `AutoCloseable` 放在 `resource` 语句块
  * 当 `try` 块执行结束后，会自动执行其 `close()` 方法，释放资源

  ```Java 
  try( /* resource 块，在此初始化只能是 实现了 Closeable 或 AutoCloseable 的对象 
  		必须是变量声明形式
  		当 try 块执行结束后，会自动执行其 close() 方法，释放资源 */ ）
  {
    	//to do
  } catch(Exception e) {
    e.printStackTrace();
  }
  ```

#### TCP

* `Socket`  `ServerSocket`

  * `Socket`
    * 通信
  * `ServerSocket`
    * 建立连接

* 建立客户端和服务端

* 通过 `Socket` 的流进行传输通信

* 必须建立连接后才能通信

* 客户端

  1. 启动客户端 `Socket` 服务，传入要连接到服务端的 ip 和 port，处理 `UnknownHostException`

     若服务端未启动，客户端进行连接，会抛异常 `java.net.ConnectException: Connection refused`

  2. 连接建立完成，拿到 `Socket` 流对象，`out - 发送`，`in - 接收`，处理 `IOException`

  3. 发送数据，处理 `IOException`

  4. 关闭 `Socket`，会自动关闭 `Socket` 流，处理 `IOException`

     若使用 `try-with-resource` 语句会自动关闭声明在 `resource` 块中的流

```Java
public class TcpSender {
    public static void main(String[] args) throws IOException {
        // 1. 启动客户端 Socket 服务，传入要连接到服务端的 ip 和 port
      	//    处理 UnknownHostException
        //    若服务端未启动，客户端进行连接，会抛异常
      	//    java.net.ConnectException: Connection refused
        Socket client = new Socket("192.168.3.42", 3456);
        // 2. 连接建立完成，拿到 Socket 流对象，out - 发送， in - 接收，处理 IOException
        OutputStream out = client.getOutputStream();
        // 3. 发送数据，处理 IOException
        out.write("Hello ServerSockert.".getBytes());
        // 4. 关闭 Socket，会自动关闭 Socket 流，处理IOException
        //    若使用 try-with-resource 语句会自动关闭 Socket
        client.close();
    }
}
// try-with-resource	
public class TcpSender {
    public static void main(String[] args) throws IOException {
        try(Socket client = new Socket("192.168.3.42", 3456)) {
            OutputStream out = client.getOutputStream();
            out.write("Hello ServerSocket".getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

* 服务端

  1. 启动服务器 `ServerSocket` 服务，监听 port，处理 `IOException`

  2. 阻塞等待客户端连接请求，拿到客户端 `Socket` 对象

  3. 通过客户端 `Socket` 拿到 `Socket` 流对象，out - 发送，in - 接收，处理 `IOExcepiton`

  4. 接收数据，处理 `IOExcepiton`

  5. 关闭 `ServerSocket`，会自动关闭 `Socket` 流，处理 `IOEception`

     若使用 `try-with-resource` 语句会自动关闭声明在 `resource` 块中的流

     一般情况下，服务端不会关闭

```Java
public class TcpReceiver {
    public static void main(String[] args) throws IOException {
        // 1. 启动服务端 ServerSocket 服务，监听 port，处理 IOException
        ServerSocket server = new ServerSocket(3456);
        // 2. 阻塞等待客户端连接请求，拿到客户端 Socket 对象
        Socket client = server.accept();
        // 3. 通过客户端 Socket 拿到 Socket 流对象，out - 发送， in - 接收，处理 IOException
        InputStream in = client.getInputStream();
        // 4. 接收数据，处理 IOException
        byte[] buffer = new byte[1024];
        int len = in.read(buffer);
        System.out.println(new String(buffer, 0, len));
        // 5.  关闭 ServerSocket，会自动关闭 Socket 流，处理IOException
        //     若使用 try-with-resource 语句会自动关闭 Socket
        //     一般情况下，服务端不会关闭
        server.close();
    }
}
// try-with-resource
public class TcpReceiver {
    public static void main(String[] args) {
        byte[] buffer = new byte[1024];
        try(ServerSocket server = new ServerSocket(3456);
            Socket client = server.accept()) {
            InputStream in = client.getInputStream();
            int len = in.read(buffer);
            System.out.println(new String(buffer, 0, len));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

* 文件上传
  * 阻塞方法，发送上传成功反馈时，要防止互相等待
  * 客户端在文件发送完成后
    * 自定义结束标识，服务端读到时，停止循环，有缺陷，若文件中含有结束标识容易容易出错
    * 发送完后，`socket.shutdownOutput()` ，发送一个正常结束序列，服务端收到后脱离阻塞状态
  * 开启多线程文件上传
* **文件上传 write 方法一定要 flush()**
  * **用了缓冲流，一定要 flush()**

# Java Reflection

#### 类加载器 *ClassLoader*

* **类加载**

  当程序要使用某个类时，如果该类还没有加载到内存中，则会通过以下三步加载

  * 加载
    * 将 class 文件读入内存中，并**创建一个 Class 对象** （字节码文件对象）
  * 连接
    * 验证：确保被加载类的正确性（验证 class 文件格式合法性）
      * `magic number` ：`0xCA 0xFA 0xBA 0xBE` （会先检查 class 文件前 4 个字节）
    * 准备：为类的静态成员分配内存，并设置默认初始化值
    * 解析：将类的符号引用（编译方面）替换为直接引用
  * 初始化
    * 静态代码块

* 类加载时机

  * 创建类的实例
  * 访问类的静态变量
  * 调用类的静态方法
  * **通过反射方式强制创建某个类或接口对应的 `java.lang.Class` 对象**
  * 初始化子类，子类父类都会加载
  * 直接使用 `java` 命令运行某个主类（public class）

* **类加载器**

  * 负责将 `.class` 文件加载到内存中，并为之生成 Class 对象

* 组成

  * 根类加载器 *Bootstrap ClassLoader*
    * 也称引导类加载器
    * 负责 Java 核心类库加载，`/lib/rt.jar`
  * 扩展类加载器 *Extension ClassLoader*
    * 负责扩展类库加载，`/lib/ext/` 
  * 系统类加载器 *System ClassLoader*
    * 应用类加载器
    * 开发者自定义的类

#### 反射 *reflection*

* 运行状态中，动态获取任意一个类所有属性或调用任意方法
* 前提：拿到 **Class 字节码文件对象**

#### 获取字节码文件对象

* 方式一：`Object`  `getClass()`
* 方式二：`类名.class`  类的字面值常量
  * 注：虽然触发了类加载，但只执行了 加载、连接 过程，没有执行 初始化 操作
  * 即：触发类加载，但静态代码块没有执行
* **方式三**：**`Class.forName(String name)`**   name 必须是全类名：`包名 + 类名`
  * 处理 `ClassNotFoundException`
  * 注：**推荐**使用该方式，灵活，可以通过参数控制类加载

```Java
// 获取字节码文件对象
public class GetClassObject {
    public static void main(String[] args) throws ClassNotFoundException {
        // 方式一：Object getClass() 方法
        Person p = new Person();
        Class aClass = p.getClass();
        System.out.println(aClass.getName());
        // 方式二：类名.class 类的字面值常量
        // 注：虽然触发了类加载，但只执行了 加载，连接 过程，没有执行 初始化
        // 即：触发类加载时，静态代码块没有执行
        Class bClass = Person.class;
        System.out.println(bClass.getName());
        // 方式三：Class.forName(String name) name 必须是全类名：包名 + 类名
        // 处理 ClassNotFoundException
        // 注：推荐使用该方式，灵活，通过传入参数控制类加载
        Class cClass = Class.forName("day28.net.Person");
        System.out.println(cClass.getName());
    }
}
class Person {}
```

#### 一个类(Class)：构造方法(Constructor)，成员变量(Field)，成员方法(Method)

#### Class API

* `getConstructors()`
  * 处理 `NoSuchMethodException`
  * 返回该类所有 `public` 构造器，数组
* `getDeclaredConstuctors()`
  * 处理 `NoSuchMethodException`
  * 返回该类所有构造器，数组
* `newInstance()`
  * 处理 `InstantiationException`
  * 调用无参构造方法实例一个该类对象
* `getConstructor(Class<?> … parameterTypes)`
  * 处理 `NoSuchMethodException`
  * 返回一个符合方法签名（形参类型）的 `public` 构造器
* `getDeclaredConstructor(Class<?> … parameterTypes)`
  * 处理 `NoSuchMethodException`
  * 返回一个符合方法签名（形参类型）的任意访问修饰符的构造器
  * `private` 修饰符的方法仍然无法访问，无法通过 `private` 构造器实例对象

#### Constructor API

* `newInstance(Object … initargs)`
  * 处理 `InvocationTargetException`
  * 根据获取的当前方法签名构造器，传入初始化参数，初始化一个该类实例
* `setAccessible(boolean flag)`
  * 处理 `IllegalAccessException`
  * **true** 取消访问权限检查，**private** **可以访问以及构造实例**


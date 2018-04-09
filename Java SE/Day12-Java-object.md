# Java Object 类

#### 内容摘要

匿名内部类，链式调用，内部类补充，Object 类，String类，

------

#### 匿名内部类

* 局部内部类的简化写法
* 前提：存在一个抽象类或接口
* 本质是一个继承了类或实现了接口的**子类匿名对象**
* `new className/interfaceName() { // Override method}.method();` 
  * 创建子类匿名对象
  * 重写方法
  * 调用该方法

####链式调用

* 方法返回类型为当前类型，调用时 `return this`

* `test.method().method().method();`

#### 内部类补充

* 空接口，没有定义方法的接口，**标记作用**

* **接口引用指向实现类实例，多态，只能调用接口成员方法，看不到实现类全部**

  > 一个类，一旦实现了接口

* 内部类可以实现实质上的多重继承：多个成员内部类继承多个类

  * 形式上的多重继承：继承一个类，实现多个接口
  * JDK 8 以后，通过接口中的默认方法同样可以实现多重继承

  ```Java
  class A {
    int a;
  }
  class B {
    int b;
  }
  // 多个成员内部类继承多个类，只能使用默认修饰成员内部类
  class Main {
    class AA extends A {} // 通过访问内部类成员变量方式，实现继承多个父类
    class BB extends B {} // 调用多个父类的成员只需指定哪一个内部类去调用
  }
  ```

* 多个接口有同名抽象方法，同时实现这几个接口，普通类只能重载一个方法，导致无法区分调用

  * 解决方案：用多个成员内部类实现这几个接口，分别重载方法
  * 详见 `day12.javaobject.InnerClassTest`

  ```Java
  interface A {
    void show();
  }
  interface B {
    void show();
  }
  // 实现多个接口，同名抽象方法，只能使用默认修饰成员内部类
  class Main {
    class AA implements A {
      public void show() {
        // A interface to do
      }
    }
    class BB implements B {
      public void show() {
        // B interface to do
      }
    }
    // 使用时，需要指定具体实现接口内部类的覆盖方法
  }
  ```



#### Object 类

* 类层次结构的根类

* 所有类都直接或间接的继承该类

* 默认无参构造方法： `public Object(){}`

* 除了 `equals` 方法是普通方法，其他方法都是`native` 方法

* ` public final class getClass()`

  * 返回 当前类 运行时 Class 类，即字节码文件内容的对象表示
  * `Class 类` **反射**见后 
    * 实例表示正在运行的 Java程序 的类和接口
    * `getName()`   `getSimpleName()`

* `public String toString()`

  * 返回该对象的字符串表示 `全类名@哈希码无符号十六进制`

  * 如 `com.sample.www.Test@14ae2e4`

  * 建议所有子类都重载该方法

    ```Java
    // Object toString source
    public String toString() {
      return getClass().getName() + '@' + Integer.toHexString(hashCode());
      // 全类名 @ 哈希码无符号十六进制
    }
    ```

* `public int hashCode()`

  * 返回该对象的哈希码值
    * y = f(x)，函数 f 即哈希映射
    * 哈希码值 = 哈希映射（对象内存地址）
    * 将该对象的内存地址通过哈希映射计算为一个整数
  * 哈希码值是内存地址的一种形式
  * 常规协定
    * 哈希码 的生成与对象中所包含的信息（对象中的成员变量）相关
    * 两个对象 equals 结果为 true，这两个对象 哈希码 相同
    * equals 结果为 false，**不要求**两个对象的 哈希码 一定不同

  > The general contract of `hashCode` is:
  >
  > * Whenever it is invoked on the same object more than once during an execution of a Java application, the `hashCode` method must consistently return the same integer, provided no information used in `equals`comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.
  > * If two objects are equal according to the `equals(Object)` method, then calling the `hashCode` method on each of the two objects must produce the same integer result.
  > * It is *not* required that if two objects are unequal according to the `Object.equals(java.lang.Object)`method, then calling the `hashCode` method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

* public boolean equals(Object obj)`

  *  `==` 比较内存地址
  *  指示其他某个对象是否与此对象相等
  *  默认实现：比较的是两个对象的地址值，即 ` this == obj` 引用变量的值
  *  重载实现：根据实际比较自定义
  *  对于任何非空引用值：
    * 自反性：`x.equals(x) ` // true
    * 对称性：`y.equals(x)` // true ==> `x.equals(y)`
    * 传递性：`x.equals(y)` // true `y.equals(z)` //true ===> `x.equals(z)`
    * 一致性：依据所有成员变量的比较结果
    * `x.equals(null)` // false

* `protected void finalize()`

  * 当垃圾回收器确定不存在该对象的更多引用时，由对象的**垃圾回收器调用**此方法
  * 被当作垃圾回收之前，会调用此方法，`native`方法
  * 故可以实现回收前显式调用 C++ 提供的接口销毁 C++ 相应内存空间
  * `System.gc()` 强制让垃圾回收器回收当前垃圾
  * 对于任意给定对象，Java 虚拟机**最多执行一次** `finalize` 方法
  * **不推荐**重载此方法，因为垃圾回收器**执行时间不确定**

* `protected Object clone()`

  * 创建并返回此对象的副本
  * 实现 `Cloneable` 接口，才允许复制，否则会抛异常 `CloneNotSupportedException`
  * 默认实现是一种浅拷贝：
    * 若成员变量中有引用类型，只会拷贝同一个内存地址，即指向同一个对象
    * 引用数据类型只是进行了引用的传递，而没有真实的创建一个新的对象
  * 深拷贝
    * 对引用数据类型进行拷贝的时候，创建了一个新的对象，并且复制其内的成员变量
    * 对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容
    * 方式：
      * 序列化，作浅拷贝，再反序列化得到拷贝内容
      * 每个要拷贝的引用类型都实现自己的深拷贝方法
        * 每个类知道自己包含哪些引用类型的成员变量
        * 深拷贝时对每一个引用类型调用其深拷贝方法

#### String 类 

* 多个字符组成的一串数据，字符序列

* 可以看成字符数组

* Java 中所有字符串字面值（如 “abc” ）

* 字符串是常量，其值创建后不能更改

* 区别字符串对象和字符串常量

  * `String s = new String("hello");`
    * 栈中引用地址，指向堆中对象，对象里面存的内存地址，指向常量区 `“hello”`
  * ` String s = "hello"`
    * 栈中引用地址，指向常量区 `“hello”` 

  ```Java
  String s1 = "hello";
  String s2 = "world";
  String s3 = "helloworld";

  // 字符串拼接操作中，有变量形式参与，先在堆上开辟空间，然后在空间中进行拼接，再放入常量池
  // 若存在则为结果赋同一内存地址，若不存在则开辟空间为其分配内存地址
  System.out.println(s3 == (s1 + s2)); //false
  System.out.println(s3.equals(s1 + s2)); //true

  // 两个字面值字符串常量做拼接的时候，先拼接结果，然后，在常量池中找有没有这个结果
  // 若存在则为结果赋同一内存地址，若不存在则开辟空间为其分配内存地址
  System.out.println(s3 == ("hello" + "world")); //true
  System.out.println(s3.equals("hello" + "world"));//true
  ```

#### StringBuilder

* 字符串缓冲，可变的，修改方便
* 单线程可变字符串序列（`Stringbuffer` 多线程）
* `append`
* `insert`

#### Split()

```Java
// 坑： . 在 split 属于正则表达式里，代表任何字母
// 要用 . 需要 [.]  or \\. 转义
String[] split = s.split("[.]");
```

> So, if you want to split on e.g. period/dot `.` which means "[any character]" in regex, use either [backslash `\`]to escape the individual special character like so `split("\\.")`, or use [character class `[\]`to represent literal character(s) like so `split("[.]")`, or use [`Pattern#quote()`] to escape the entire string like so `split(Pattern.quote("."))`.
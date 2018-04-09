# Java 类和对象

#### 内容摘要

final，package，import，方法覆盖，父类引用指向子类对象，父类禁止子类的措施

------

#### final 关键字

* `final` 修饰的变量不能修改其值，即自定义常量

  * `final int i` 自定义常量

    * 自定义常量注意命名规则，每个字母大写，每个单词用下划线 `_` 隔开

  * 初始化**非静态**自定义常量：

    * 声明变量时赋初值

    `final int i = 10;`

    * 构造代码块中赋值

    ```Java
    class Test {
      final int i;
      { // 初始化代码块，可以初始化 final 常量值
        i = 10;
      }
      public Test() {}
    }
    ```

    * 构造方法中赋初值，可以**动态赋值**

    ```java
    class Test {
      final int i; // 若没有显式初始化，则必须在构造方法中初始化
      public Test () { // 构造方法中，必须初始化 final 常量的值，每个构造方法需要有初始化操作
        i = 10;
      }
      public Test(int i) {
        this.i = i;
      }
    }
    ```

  * 初始化**静态**自定义常量：初始化静态自定义常量：初始化**静态**自定义常量：初始化静态自定义常量：

    * 声明变量时赋初值

    `final static int i = 10;`

    * 静态代码块赋初值

    ```Java
    class Test {
      final static int i;
      static { // 静态代码块，可以初始化 final 常量值
        i = 10;
      }
      public Test() {}
    }
    ```

  ```java
  class Test {
    final int i; // 若没有显式初始化，则必须在构造方法中初始化
    public Test () { // 构造方法中，必须初始化 final 常量的值，每个构造方
  ```

*  `final` 修饰的类不能继承

  * `private` 私有构造方法的类也不能继承

* `final` 修饰的方法不能覆盖

* 提高安全性，加快某些类型检查过程

#### package 关键字

* 必须声明在 **第一行** ，`import` 语句之前，用来指定定义的类成为 **包** 的成员
  * `package <包名>`
  * 命名规则：域名反转
* 用途
  * 维护代码：相关的类和接口分组，包名通常表示各个类和接口的用途
  * 同名类：创建了不同的命名空间（*Namespace*），解决了命名冲突问题
  * 提供了对应用程序内部实现机制的保护域
    * default 跨包不能访问
    * protected 跨包非子类不能访问
* 若没有声明 `package` ，则属于 JDK 默认包

```bash
// 带包的类手动编译与执行
src/day09/javaextends
➜ javac -d . PackageTest.java  // 编译
     // -d 设置 .class 创建文件目录
     //  . 指定当前目录和当前以下目录
     //  会自动创建包文件夹

src/day09/javaextends
➜ java day09.javaextends.PackageTest  // 运行，要带全包名
This is package compile test.
```

#### import 关键字

* 默认情况下，编译器只会在当前包下搜索类，搜索不到则类未定义，解决方法

  * 使用全类名 `包名.类名`  ，类似于绝对路径，如 `java.util.Scanner` 
  * 使用导包 `import` 如 `import java.util.Scanner;`

* 导入其他包中 public 类

  * 编译器会在 `import` 指明的包下搜索类

* 必须在类声明之前

* `java.lang` 包中的类，每个 java 文件会隐式导入

* 导入包下全部类： `import java.util.* ` ，不包括包内其他子包

  * `.*` 不能递归导入，场景如下

    ```Java
    // 目录结构
    com
    |- sample
    	|- www
    		|- Student.java // 包名：com.sample.www
    		|- insidefolder 
    			|- Teacher.java // 包名：com.sample.www.insidefolder

    import com.sample.www.*; // 仅导入该包名下所有内，而不包括文件夹

    // 所以只有能用 Student ,要使用 Teacher，需要如下声明
    import com.sample.www.insidefolder.*;

    // 解决方法
    import com.sample.www.Student;
    import com.sample.www.insidefolder.Teacher;
    ```

* 静态导入

  * 导入某个类中静态成员或静态方法
  * `import static + 静态成员名／方法名`
  * 例如：`import static java.lang.System.out`
    * 使用：`out.println("Hello");`
  * 若本类中引入` 静态成员变量／方法` 与类中`变量／方法` 同名时，会使用本类的

#### 方法覆盖 *override*

* 在子类中替换父类或祖先类对某个方法的实现
* 子类重新定义父类或祖先类有 **相同方法签名和相同返回类型** 的方法实现
  * 方法签名 *signature*：方法名 + 参数列表
  * 返回类型
    * 子类中覆盖父类方法，若返回类型是引用类型，只要类型兼容即可
    * 子类类型即是子类实例，又是父类实例，即父类类型兼容子类类型
  * **访问修饰符**
    * **子类可以保持和父类访问权限一样**
    * **可以扩大访问权限，不能缩小**
    * `priavte` => `默认` => `protected` => `public`
* **只有在子类中可以访问到父类或祖先类的方法，才可以覆盖**
  * 即 父类中 `private` 方法不能覆盖
* 注解 `@Override` 
  * 让虚拟机帮忙检查是否完成覆盖工作
  * **避免写错方法签名或返回类型，导致重载父类方法**
* `final` 修饰的方法不能在子类进行覆盖
* `synchronized` `native` 可以在覆盖方法中自由使用
* 注意事项
  * 父类方法参数列表中被声明为 `final` 的形式参数，子类覆盖方法后可以不用 `final` 修饰形式参数
  * 子类覆盖方法声明的异常列表中的异常类，必须与父类异常列表的异常类兼容
  * **静态方法不能被覆盖，只能隐藏**

#### 父类引用指向子类对象

* `Father father = new Son();`
* 父类引用调用父类方法时，具体执行

#### 多态 *polymorphism*

* 同一个事物（相同方法名），在不同时空表现出来的不同形态（不同方法体）
* 前提：
  * 继承关系
  * 方法覆盖
  * 父类引用指向子类对象

#### 父类禁止子类访问自己成员变量、调用成员方法的措施

1. `private` 成员变量或方法，对子类隐藏
2. 子类与父类异包，父类中成员变量修饰符为 `默认`，子类无法访问

#### 子类不能覆盖父类方法

1. `fanal`
2. `private`
3. `static`
# Java Exception 类

#### 内容摘要

可变参数列表，装箱和拆箱，Exception 类，try catch，throwable，throws / throw，finally，自定义，时间 API

------

#### 可变参数列表

* 参数个数灵活可变 `参数类型... 参数名`

  ```java
  public static void test(int... args) {}
  public static void demo(String... args) {}
  public static void toDo(){}
  public static void exam(int i, boolean... bool){} // 见 3. 有且只有1个，放在最后
  public static void method(int[] array){}// 见 4. 参数列表重复，无法重载
  /*
      1. 可以接受多个参数
      2. 可以接受一个参数
      3. 可以不传入参数
      args 可以直接当做数组来处理
  */
  public static void main(String[] args) {
    // 见 1. 不传参数时，2个可变参数列表无法调用
    test();
    demo();
    
    // 见 2. 会优先重载调用 todo()，而不会执行 test()
    todo();
    test();
    
    
    test(1);
    test(1, 2, 3, 4, 5, 6);
  }
  /*
  	重载问题：
  	1. 当不传参数的时候可能会与其他可变参数列表方法混淆，报错，无法确定
  	2. 编译器在重载时，会优先查找参数列表确定方法
  	3. 在参数列表中，可变参数列表只能定义一个且放在最后位置
  	4. 对于编译器，可变参数列表也被当做数组对待，重载数组参数会失败
  */
  ```

#### 装箱和拆箱

* 将原始类型值转自动地转换成对应的对象
* 每一种基本数据类型都有其对应的包装类
  * `Integer` `Boolean` `Double` `Float` `Long` `Byte` `Short` `Character`
* 拆箱：包装类型的值可以直接赋值给基本数据类型，去掉包装类型
* 装箱：基本数据类型的值可以直接赋值给包装类型
* 语法糖：虚拟机会辅助完成这些操作
  * 比如 ` int i = integer.intValue()`  ` Integer i = integer.valueOf(10)`
  * 写法简便，但性能无影响

```java
Integer integer = new Integer(10); // 直接声明，麻烦

int i = integer; // 去掉包装，拆箱

int j = 10;
Integer integer = j; // 包装赋值，装箱
```

* 面试注意：JVM会缓存-128到127的Integer对象，若 Integer 比较大小，其实是指向同一对象，内存地址相同

  > 在Java中，会对-128到127的Integer对象进行缓存，当创建新的Integer对象时，如果符合这个这个范围，并且已有存在的相同值的对象，则返回这个对象，否则创建新的Integer对象。

  ```Java
  Integer i = 100;
  Integer j = 100;
  Integer k = 200;
  Integer l = 200;
  System.out.println("i == j = " + (i == j)); // true 指向同一对象，内存地址相同
  System.out.println("k == l = " + (k == l)); // false 两个不同对象，内存地址不同
  ```

  ​

#### Exception 类

* `Throwable` 所有异常／错误的超类
  * `Error ` 非常严重的异常情况，自己无法处理
    * 内存溢出等
  * `Exception` 可以在程序中解决的异常情况，可以捕获的异常，手动处理
    * `Exception` 及其子类，编译时异常，可以预见 checkable
      * 编译器会强制要求提前做好处理准备 try catch
    * `RuntimeException` 运行时异常，不容易预见
      * 如空指针，类转换异常
      * 编译器可能无法预见，不会强制要求 try catch
* 当发生异常时，默认执行流程
  1. 从异常发生处，终止代码执行
  2. 虚拟机执行应对异常代码，捕获异常，获取异常对象
  3. 将异常信息输出到控制台
* 手动捕获处理异常

#### try / catch

* 一个 try ，可以对应多个 catch 分支，但只会执行其中一个 catch 语句块
* 若要捕获的异常类型出现兼容情况，要注意书写顺序，先把**能确定的具体的异常写在前面**，不确定的写在后
  * 没有继承关系的异常，顺序无所谓
  * 若有继承关系，父类型写在子类型之后
* 执行流程：
  1. 在 try 块发生异常时，由虚拟机捕获异常，构造异常对象
  2. 根据发生异常的具体异常类型，匹配 catch 中的异常参数
  3. 在所匹配到 cath 块中执行处理代码
  4. 若 catch 语句中没有能匹配的异常参数，虚拟机会执行默认异常处理

```Java
try {
  // 可能发生异常的代码
  // 发生异常之后的代码不会执行
  // 一旦发生异常就转到 catch 语句 
} catch (Exception e) { // 捕获异常，可以捕获具体异常类型
  // 处理异常
} finally {
  
}

try {
  
} catch (...) {
  
} catch (...) {
  
} catch (...) {
  
}

// JDK 7 new features
try {
  
} catch (ArrayIndexOutOfBoundsException | NullPointerException e) {
  // 若两个异常都要写相同处理逻辑
}
```

####  Throwable 

* 获取异常对象信息


* `getMessage`
* `toString()`
* `printStackTrace`

#### throws / throw

* `throws`
  * 将异常的发生和异常的处理分离开来，抛给方法的调用者
    * 当发生异常时不知该如何去处理时，抛给上一级调用者去处理
    * 从语法角度，**强制**调用者来处理调用方法时可能抛出的异常，
  * **只针对可检查异常即编译时异常**
  * 在**方法头部**，方法或构造函数，throws 抛出**异常类型**
  * `<返回类型> <参数列表> throws <异常列表> {}`
  * 可以声明抛出多种异常，一旦声明，需要全部捕获
* `throw`
  * 在**方法体**中 throw 抛出**异常类对象**
  * `throw <异常类对象>`  // 可以通过构造方法传入异常类 message 内容
  * 可以抛出编译时异常和运行时异常，当抛出编译时异常时需要和 throws 配合使用


| throws               | throw              |
| -------------------- | ------------------ |
| **方法后**，跟**异常类名**    | **方法体内**，跟**异常对象** |
| 跟**多个**异常类名，由方法调用者处理 | 一次只能抛出**一个**异常对象   |
| **可能**抛出异常，但不一定会抛    | **必然**抛出异常         |

#### 处理原则

* 方法内部可以处理，则 try catch
* 处理不了，则 抛出 throws throw

#### finally

* **finally 代码块一定会执行 ，不管是否抛出异常**
* 一种特殊情况：走到 finally 之前 ，虚拟机退出，finally 代码块不会执行
* 面试题：
  * 如果 catch 内有 return 语句，finally 里代码还会执行吗，执行顺序？
  * **return 在栈帧中有个单独的空间**
  * 还会执行，顺序是先 finally 再 return

```Java
try {
  
} catch (Exception e) { 
  
} finally {
  // 释放资源操作
}

try {
  throw new RuntimeException();
} catch (RuntimeException e) {
  System.out.println("catch hello");
  return "catch";
} finally {
  System.out.println("finally hello");
  return "finally";
}
// 返回 "finally"
```

#### 自定义异常

* 继承 `Exception` ，虚拟机会将自定义异常当做编译时异常

#### 异常处理机制

* 终止模型
* 恢复模型

```Java
public class RecoveryModel {
    public static void main(String[] args) {
        // 假设 数组 越界异常
        int x = 5;
        int[] arr = {1, 2};
        while(true) {
            try {
                System.out.println(arr[x]);
                break;
            } catch (ArrayIndexOutOfBoundsException e) {
                System.out.println("Array index out of bounds exception.");
                System.out.println(e.toString());
                x --;
            } finally {
                System.out.println("x = " + x);
                System.out.println("\nRecovering...");
            }
        }
        System.out.println("Recovery finish...");
    }
}
```

#### 与时间有关的类 API

* `Date`
* `SimpleDateFormat`
* `Calendar ` 推荐








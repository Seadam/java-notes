# Java Reflection

#### 内容摘要

Class，Constructor，Field，Method，注解，类，元注解，定义注解，使用注解，注解处理器，配置文件，注意

---

#### 一个类(Class)：构造方法(Constructor)，成员变量(Field)，成员方法(Method)

#### Class API

* `getConstructors()`
  * 返回该类所有 `public` 构造器，数组
  * `java.lang.NoSuchMethodException`
* `getDeclaredConstuctors()`
  * 返回该类所有构造器，数组
  * `java.lang.NoSuchMethodException`
* `newInstance()`
  * 调用无参构造方法实例一个该类对象
  * `java.lang.InstantiationException`
* `getConstructor(Class<?> … parameterTypes)`
  * 处理 `NoSuchMethodException`
  * 返回一个符合方法签名（形参类型）的 `public` 构造器
* `getDeclaredConstructor(Class<?> … parameterTypes)`
  * 处理 `NoSuchMethodException`
  * 返回一个符合方法签名（形参类型）的任意访问修饰符的构造器
  * `private` 修饰符的方法仍然无法访问，无法通过 `private` 构造器实例对象
* `getFields()`
  * 返回**本类、接口和父类中**所有 public 成员变量，数组
  * 若没有 public 成员变量，或者该 Class 是一个数组类，基本类型，void 类型，则返回长度为 0 的数组
* `getDeclaredFields()`
  * 返回**本类**任意访问权限，或父类 public 的成员变量，数组
  * 不包括父类或接口的成员变量
* `getField(String name)`
  * 返回指定 name 简称**本类及所有父类中**的 public 成员方法
  * 查找算法：先在本类查找，再递归查找父类


* `getDeclaredField(String name)`
  * 返回指定 name 的**本类**任意访问权限，或父类 public 的成员变量
  * 不包括父类的成员变量
  * 如果该类是一个数组类，不能拿到数组的 length 成员变量
* `getMethods()`
  * 返回**本类、接口和父类**所有 public 成员方法，数组
  * 若没有 public 成员方法，或者该 Class 是一个数组类，基本类型，void 类型，则返回长度为 0 的数组
* `getDeclaredMethods()`
  * 返回**本类**所有访问权限的成员方法，数组
  * 不包括父类的其他成员方法
* `getMethod(String name, Class<?> ... parameterTypes)`
  * 返回**本类和接口**指定方法名 name，符和该参数类型的 public 成员方法
  * 查找算法：如果在本类中未找到，则向上递归到父类中查找
* `getDeclaredMethod(String name, Class<?> ... parameterTypes)`
  * 返回**该类**指定方法名 name，符和该参数类型的 任意访问修饰符的，或父类 public 成员方法
  * 不包括父类的其他成员方法
* `getAnnotation(Class<T> annotationClass)`
  * 返回当前 Class 对象前 注解名为 annotationClass 注解对象
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`
  * 判断当前 Class 对象前 是否有注解名为 annotationClass 的注解
* `<clint>`
  * 对象构造器
  * 不能作为方法名传入 `getMethod` 或 `getDeclaredMethod` 中，会抛异常
  * 作为构造，用来对非静态成员（包括构造代码块）初始化
* `<client>`
  * 类构造器
  * 不能作为方法名传入 `getMethod` 或 `getDeclaredMethod` 中，会抛异常
  * 作为构造，用来对静态成员（包括静态代码块）初始化

#### Constructor API

* `newInstance(Object … initargs)`
  * 处理 `InvocationTargetException`
  * 根据获取的当前方法签名构造器，传入初始化参数，初始化一个该类实例
* `setAccessible(boolean flag)`
  * 处理 `IllegalAccessException`
  * **true** 取消访问权限检查，**private** **可以访问以及构造实例**
* `getAnnotation(Class<T> annotationClass)`
  * 返回当前 Constructor 对象前 注解名为 annotationClass 注解对象
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`
  * 判断当前 Constructor 对象前 是否有注解名为 annotationClass 的注解

#### Field

* `get(Object obj)`
  * 处理 `IllegalAccessException`
  * 返回指定对象 obj 上 此 Field 对象 表示字段的值
  * 如果该字段是基本数据类型，自动包装到一个对象中
  * 若获取静态成员变量的值，则所传参数为 null `get(null)`
* `set(Object obj, Object value)`
  * 将指定对象 obj 上 此 Field 对象 表示字段的值修改为 value 
* `setAccessible(boolean flag)`
  * 处理 `IllegalAccessException`
  * **true** 取消访问权限检查，**private** **可以访问**
* `getAnnotation(Class<T> annotationClass)`
  * 返回当前 Field 对象前 注解名为 annotationClass 注解对象
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`
  * 判断当前 Field 对象前 是否有注解名为 annotationClass 的注解

#### Method

* `invoke(Object obj, Object ... args)`
  * 处理 `InvocationTargetException`
  * 指定对象 obj 传入实际参数 args ，调用此 Method 对象方法
  * 静态方法传入 null
  * 可以接受一个返回值，也可以直接调用
  * 如果返回值为基本类型，将会装箱
  * 如果是基本类型数组，则不会装箱，会返回基本类型数组
  * 如果是 void，返回 null
* `setAccessible(boolean flag)`
  * 处理 `IllegalAccessException`
  * **true** 取消访问权限检查，**private** **可以调用**
* `getAnnotation(Class<T> annotationClass)`
  * 返回当前 Method 对象前 注解名为 annotationClass 注解对象
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`
  * 判断当前 Method 对象前 是否有注解名为 annotationClass 的注解

# Java Annotation

#### 注解 *Annotation*

* 添加 **额外信息（元数据：描述数据的数据）** 的一种方式
* 在编译或运行时使用额外信息（数据）
* `@Override`  方法覆盖
* `@Deprecated ` 过时方法

#### 类

* 三种状态
  * java 源文件
  * class 字节码文件
  * jvm 字节码文件对象
* 三要素
  * 类
  * 成员方法
  * 成员变量

#### 元注解

描述注解的注解

* **`@Rentention`  ： 控制注解存活周期**
  * `RetentionPolicy.SOURCE`
    * 在 java 文件有效
    * 如：`@Override`
  * `RetentionPolicy.CLASS`
    * 编译时注解，在 `class` 文件有效
    * 如：自动生成代码，`java` 中与编译器交互的 API
  * `RetentionPolicy.RUNTiME`
    * 在运行时有效， JVM 运行时字节码文件对象
    * 如：反射实现的注解处理器
* `@Target`  ：指定该注解使用位置
  * `ElementType.TYPE`
    * 类、接口前使用
  * `ElementType.METHOD`
    * 成员方法前
  * `ElementType.FIELD`
    * 成员变量前
  * 具体查看 枚举 `ElementType`

#### 定义注解

* `public @interface 注解名 { 定义体 }`
* `@` 必不可少
* 默认自动实现  `java.lang.annotation.Annotation`  接口
* **不能继承其他注解或接口**
* 类似于接口的方法声明：`返回值类型 方法名()`  对应键值对  `数据类型  数据名`  `int min();`
  * **数据类型：基本类型，Class，Enum，String，Annotation**
  * 注解可以嵌套
* 方法后可以用 `default` 定义默认值  `int min() default 25;`
* 任意一个注解在使用时，都必须有确定的值
* 自定义注解定义完后，需要自己的注解处理器才能运行

```Java
// 定义
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface RangeCheck {
    int value() default 100;
}

// 使用
@RangeCheck(250)
public class Test {}
```

#### 注解使用

* 当健（方法名）为 `value` 并且只需要给 `value` 赋值时，可以省略 `value =` 
* 注解一旦声明，每个字段必须有确定值，**不能赋值为 null**

#### 注解处理器

* 通过反射
* `getAnnotation(Class<T> annotationClass)`
  * 返回当前 Class、Field、Method 对象前 注解名为 annotationClass 注解对象
* `isAnnotationPresent(Class<? extends Annotation> annotationClass)`
  * 判断当前 Class、Field、Method 对象前 是否有注解名为 annotationClass 的注解

#### 注解 VS 配置文件

* 注解
  * 直观，开发效率高
  * 硬编码，修改之后需要重新编译运行
* 配置文件
  * 可配置，不用改源码
  * 不直观，开发效率低

#### 注意事项

* 注解完全依赖于反射
* 防止程序设计过多配置时可以考虑使用注解


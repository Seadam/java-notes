# Java Memory Management

内容摘要

JMM，程序计数器，虚拟机栈，本地方法栈，堆，方法区，垃圾回收，引入，简介，引用计数法，根搜索算法，标记清除算法，GC 触发时机，内存溢出问题，内存泄漏，查漏补缺

---

## JMM *Java Memory Model* 

![](http://gityuan.com/images/jvm/stack_heap_info.png)

[source](http://gityuan.com/2016/01/09/java-memory/)

#### 程序计数器 *Program Counter Register*

* 一块较小的内存空间
* 当前线程所执行的字节码文件的**行号指示器**
  * JVM 指令：8bit 二进制 指令
* **线程私有**的，线程创建时创建，线程结束时释放内存空间

#### Java 虚拟机栈 *Java Virtual Machine Stacks*

* Java 方法执行的内存模型
* 每一个方法执行会创建一个栈帧（*Stack Frame*）
* 栈帧包含：局部变量表，操作栈，动态链接，方法出口（return空间）
* **线程私有**的，线程创建时创建，线程结束时释放内存空间
* 为虚拟机执行 **`java`** 方法服务

#### 本地方法栈 *Native Method Stacks*

* 同上
* 为虚拟机执行 **`native`** 方法服务

#### 堆 *Heap*

* 唯一功能**存放对象实例**
  * 类加载器加载了类文件后，把 **类（字节码文件对象）** 放入堆中，以便执行器执行
  * **字节码文件对象**，存在堆中
* 一个 JVM 实例只存在一个堆，堆的大小可以调节
* **线程共享**

#### 方法区 *Method Area*

* 存放已被虚拟机加载的**类信息 (字节码文件）**，常量，静态变量，即时编译器编译后的代码（JVM 指令）等
* **线程共享**

## 垃圾回收 *Garbage Collection*

#### 显式内存管理 ／ 自动内存管理（隐式）

* 显式内存管理（C ／ C++）
  * `malloc`   `free`  调用 API 来管理内存
  * 内存管理是程序员的职责
  * 内存泄漏问题：内存已申请，使用完毕后没有主动释放，会一直占用内存
  * 野指针：使用了一个指针，但是该指针指向的内存空间已经被 `free`
* 自动内存管理（Java ／ C# ／ 一些脚本语言）
  * 内存空间由 **垃圾回收器** 自动管理程序
  * 优点
    * 增加了程序可靠性
    * 减少了 内存泄漏（*memory leek*） 情况
    * 提高程序员效率
  * 缺点
    * 程序员无法控制 垃圾回收 时间
    * 判断哪些内存需要回收需要耗费系统开销
    * 逻辑上内存泄漏依然存在（代码问题）

#### 简介

* 工作原理：找出不再使用的对象（*garbage*），进行回收（*collection*）
* 所有不再使用的代码都是垃圾（**没有任何引用变量指向的对象**）
* 常见判断方法：引用计数法、根搜索算法，标记清除算法

#### 引用计数法

* 给堆中每一个对象增加一个引用计数器，当每一个创建一个对象并赋值给一个变量时，引用计数器 +1，当对象不在使用时（离开作用域），引用计数器 - 1，一旦该对象引用计数器为 0，就满足回收条件

* 特点：实现简单

* 缺点：**无法解决循环引用问题**

  *  A对象 和 B对象 都为垃圾时，A对象 引用指向 B对象，B对象 引用指向 A对象

     A对象 和 B对象 引用计数器为 1，不满足回收条件

#### 根搜索算法 （标记算法） *Root Tracing*

* 通过一系列名为 GC Root 的对象作为起始点，从这些节点向下搜索，搜索路径称为引用链(*Reference chain*)，当一个对象所有 GC Root 之间没有任何引用链相连（图论说法：GC Root到这些对象都不可达时），证明该对象是不可用的，满足回收条件，进行回收
  * 通过引用变量，找得到就不是垃圾，找不到就是垃圾


* 主流商用程序处理语言 （*hotswap*）， 使用此方法
* GC Root
  * 虚拟机栈中的引用对象
  * 方法中静态属性引用的对象
  * 方法区常量引用的对象
* `shallow size`
  * 对象本身占用内存大小
  * 即对象头加成员变量占用内存大小总和
* `retained size`
  * 回收一个对象最多能被 GC 回收的内存总和
  * 该对象本身的 `shallow size` 加上仅可以从该对象访问（直接或间接）的对象的 `shallow size` 之和

#### 标记清除算法 *Mark Sweep Alogrithm*

* JVM 和 Dalvik （Android 虚拟机）使用此算法实现 GC （Android 5.0 之前的虚拟机 之后 ART）
* `mark`  ：标记出被引用对象
  * 根搜索算法作标记
* `sweep`  ：清楚那些没有任何引用的对象
* 缺点：回收空间后会产生内存碎片
* **复制算法**：内存一分为 2，使用一半，备用清除一半
  * 可以避免内存碎片
  * 缺点：内存只能用一半

#### JVM 堆内存管理

* 新生代
  * 死得快（匿名内部类对象），使用**标记清除法**
* 老年代
  * 内存空间占用大，死得慢，使用**复制算法**

#### GC 触发时机

* 申请堆内存（heap space）失败后会触发
* 系统进入 空闲（idle）后一段时间后会触发
* 主动调用 触发 `System.gc()`

#### 内存溢出问题 *Out of Memory*

* 堆溢出 *Heap OOM*
  * 若虚拟机栈可以动态扩展，当扩展无法申请到足够的内存会抛出
* 栈溢出 *Stack Overflow*
  * 线程请求的栈深度大于虚拟机允许的深度，抛出该异常

#### 内存泄漏

* 可用内存少于物理内存
* 内存泄漏可能会导致内存溢出（堆溢出）


## 查漏补缺

```Java
public class Test {
    public static void main(String[] args) {
        Father son = new Son(1000);
    }
}

class Father {
    int i = 10;
    public Father() {
        System.out.println(getI());
    }
    public int getI() {
        return i;
    }
}

class Son extends Father {
    int i = 100;
    public Son(int i) {
        this.i = i;
    }
    @Override
    public int getI() {
        return i;
    }
}
```

* 输出 0
* 子类构造前会先调用父类构造方法 `Father()`
* 父类构造方法中调用的是子类的覆盖方法 `getI()`
* 由于子类还为初始化成员变量，所以返回 0
# Java IO Stream

#### 内容摘要

概述，结构，分类，字节流写数据，字节缓冲流，两者比较，字符流，编码表

------

#### 概述

* IO流用来处理设备之间的数据数据传输
* Java 对数据的操作是通过流的方式

#### Stream Structure

* `InputStream`
  * `FileInputStream`
  * `BufferedInputStream`
  * `System.in`
  * `ObjectInputStream` 
* `OutputStream`
  * `FileOutputStream`
  * `BufferedOutputStream`
  * `FilterOutputStream`
    * `PrintStream`
  * `ObjectOutputStream ` 
* `Writer`
  * `InputStreamWriter`
    * `FileWriter`
  * `BufferedWriter`
  * `PrintWriter`
* `Reader`
  * `OutputStreamReader`
    * `FileReader`
  * `BufferedReader`

#### 分类

* 数据流向
  * 输入流：读入数据  input
    * （文件）外部设备 ====>  内存（JVM）
  * 输出流：写出数据  output
    * 内存（JVM）====>  外部设备（文件）
  * 以虚拟机为参照
    * 流向 JVM —— 输入
    * JVM 流出 —— 输出
* 数据类型
  * 字节流 抽象类
    * `InputStream`
    * `OutputStream`
  * 字符流 抽象类
    * `Reader`
    * `Writer`
* **字节流适于所有数据，如图片、视频**
* **字符流能对文本数据操作方便**

#### 字节流读写数据

* `OutputStream` 抽象类
  * `close()`
    * 关闭流，并释放系统资源
    * 垃圾回收器不能直接回收系统资源，所以要手动关闭
  * `flush()`


* `FileOutputStream`

  * 和 File 对文件路径的抽象表示不同，通过文件路径创建 `FileOutputStream` 对象
    * 若文件不存在，则会根据文件路径创建一个**物理存在的文件**再与 `FileOutputStream` 对象相关联
  * 构造方法
    * `FileOutPutStream(File file)`
      * 传入指定文件，即要输出的文件路径
      * 同下
    * `FileOutPutStream(String name)`
      * 在一次写入过程中，每次写入会自动拼接到文件末尾
      * 传入指定文件路径
      * 若文件存在，则直接建立通道
      * 若文件不存在则会根据文件路径创建一个物理存在的文件，再建立通道
      * 若是一个目录，会抛出异常 `FileNotFoundException`


  * 方法
    * `public void write(int b)`
      * 将指定字节写入此文件输出流 ，无符号位 0 ~ 255 的数值
      * `byte` 只能表示 8 位， -128 ~ 127 ，所以用 `int`
      * 把 `-1` 当作流结束的标志，要保证 -1 不是数据，所以用 `int`
    * `public void write(byte[] b)`
    * `public void write(byte[] b, int offset, int length)`
  * 换行
    * 依赖于操作体统
      * linux 或类 Unix 操作系统 ：`'\n'`
      * mac                                     :  `'\r'`
      * windows                             : `"\r\n"`
  * 追加写入
    * `boolean append`  追加到文件末尾，而不覆盖
  * 异常
    * `Closeable` 接口，所有流都实现了，所以可以多态
    * `try - cath - finally`  中关闭流 自己写方法 `closeQuietly(Closeable stream)`  方便阅读
    * 1.7 `try-with-resourse` 实现 `autoCloseable` `Closeable` 接口，会自动关闭流

* `InputStream`

  * `close()`
  * `flush()`

* `FileInputStream`

  * 构造方法
    * `FileInputStream(File file)`
      * 同下
    * `FileInputStream(String name)`
      * 如果找不到物理文件或找到目录，就会抛出异常 `FileNotFoundException`
  * 方法
    * `public int read()`
      * 一次读取一个字节
      * 返回文件数据下一个字节，若到文件末尾，返回 -1
      * 循环条件 `while ((int read = fis.read()) != -1)`
    * `public int read(byte[] buffer)`
      * 每次读取多个数据，提高效率 
        * `byte[] buffer = new byte[1024]`  缓冲容量大小 1024 的整数倍
        * 与 read 方法差距在，与操作系统交互次数少，效率高
      * 读取多个数据放入 缓冲数组 buffer 中
      * 返回读取的数据长度，由此来创建确定长度的字符串
    * `public void write(byte[] b, int offset, int length)` 
      * 文件复制时，常用

* 复制文件

  * 读取源文件 `FileInputStream`
    * `while ((int len = fis.read(buffer)) != -1)`
  * 写入新文件 `FileOutputStream`
    * `fos.write(buffer, 0, len)` // 一定要写 len ，避免打不开文件

#### 字节缓冲流

* 包装流（装饰者设计模式：给已经做好的东西添加新的功能）
* 加入了缓冲区，构造时需要传入一个 `OutputStream`  `InputStream` 底层流对象
* 关闭包装流会自动关闭底层流
* `BufferedOutputStream`
  * 构造方法
    * `public BufferedOutputStream(OutputStream out)`
      * 将数据输出到底层流
    * `public BufferedOutputStream(OutputStream out, int size)`
* `BufferedInputStream`
* 调用 `close()` 前会自动调用 `flush()`


* 复制练习

#### 缓冲与非缓冲比较 

* 执行时间：`BufferBuffer`  <  `FileBuffer`  <  `BufferByte`  <  `FileByte`

#### 补充其他知识

* `Arrays.asList(T…t)` ：将数组转换为固定长度的 List 类型，不能添加和修改操作

* `Collections.synchronizeList(ArrayList list)` ：返回一个线程安全(synchronized)的 Arraylist

* `Collections.shuffle(List list)` 打乱数组顺序

* 多线程 + 异常：

  * 主线程不能用 try catch 捕获线程中抛出的异常
  * `Thread.UncaughtExceptionHandler` 回调，可以捕获线程
  * `Thread.setDefaultUncaughtExceptionHandler`
  * 注意，要在**线程启动前声明**

  ```Java
  // 捕获线程
  public class ThreadException {
      public static void main(String[] args) {
          Thread thread = new Thread(new MyThread());
        	// 回调
          thread.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
              @Override
              public void uncaughtException(Thread t, Throwable e) {
                	// 此处打印日志文件，提供 bug 修复源头
                  System.out.println("Caught thread exception..");
              }
          });
        	thread.start();
      }
  }

  class MyThread implements Runnable {
      @Override
      public void run() {
          throw new IllegalArgumentException();
      }
  }
  ```

  ​








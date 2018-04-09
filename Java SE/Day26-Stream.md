# Java IO Stream

#### 内容摘要

结构，编码表，字符流，简化字符流，字符缓冲流，**多级缓存写入**，标准输入输出，序列化流

------

#### Stream Structure

* `InputStream`
  * `FileInputStream`
  * `BufferedInputStream`
  * `System.in`
  * `ObjectInputStream` 
* `OutputStream`
  * `FileOutputStream`
  * `FilterOutputStream`
    * `PrintStream`
    * `BufferedOutputStream`
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

#### 编码表

* 由字符及其对应的数值组成的一张表

* 编码 ／ 解码

  * 编码：字符转化为对应码表中的码值
  * 解码：字符相应的码值根据码表转化为相应的字符
  * 依赖于具体的码表
  * 乱码：编解码不一致
  * 字符串编码
    * `getBytes(CharSet charset)` // 没有指明编码的情况下，使用默认字符集进行编码
    * `CharSet.defaultCharset`
    * `CharSet` 编码表
  * 字符串解码
    * `new String(byte[] byte, CharSet charset)`

* 常见编码表

  * `ANSI` 操作系统默认字符集

  * ASCII ／ Unicode 字符集

    * ASC ii  :   `'a' = 97`    `'A' = 65`    `'0' = 48`

  * ISO-8859-1 Latin-1

    * 拉丁码表（ASC ii 的扩展）
    * 在一个字节中，ASC ii 没有使用的值拿来当做码值

  * GB2312 ／ GBK ／ GB18030

    * 以 **2个字节**来存储一个中文字符
    * 简体中文，区别在于包含中文字符个数不一样
    * 字符个数一个比一个多

  * BIG5

    * 繁体中文码表

  * UTF-8

    * [参考阅读](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
    * `Unicode`
      * 包含所有已知字符且编码值唯一的码表
      * 规定了每一个字符的码值
      * 并未规定每一个码值在计算机中的存储方式
      * 十六进制表示
    * `UTF-8`
      * 若要支持国际化，一般都会使用 `UTF-8`
      * 实现了码值的存储方式
      * 中文占 2个字节，但是**存储为 3个字节**
      * Java class 文件使用 `UTF-8` 编码
      * 变长编码，前缀编码
    * `UTF-16`
      * 也就是 `unicode` 编码
      * 用 **2个字节**来存储码值
      * JVM 中使用 `UTF-16` 编码
      * 定长编码

    ```Java
    // UTF-8 规则
    将 Unicode 0000 0000 - 0000 007F 的字符，用 1 个字节来表示
    将 Unicode 0000 0080 - 0000 07FF 的字符，用 2 个字节来表示
    将 Unicode 0000 0800 - 0000 FFFF 的字符，用 3 个字节来表示
    1 个字节 0xxx xxxx 与 ASCii 相同，兼容 ASCii 码表
    2 个字节 110x xxxx 10xx xxxx
    3 个字节 1110 xxxx 10xx xxxx 10xx xxxx

    // 注意，汉字占 2 个字节，但是存储是按照 3 个字节来存储的
    如： 假设 '汉' Unicode 表示为 AF97  => 二进制 1010 1111 1001 0111 
           1010   11 1110   01 0111
      	   ^^^^   ^^ ^^^^   ^^ ^^^^
      1110 xxxx 10xx xxxx 10xx xxxx
      	结果 UTF-8 以三个字节来存储 => 1110 1010 1011 1110 1001 0111
    ```

#### 字符流

* **字符流 = 字节流（底层流） + 编码器（解码器）** 装饰者设计模式
* 字节流操作字符不方便，中文字符占2个字符，容易出现截断操作，导致解码出错
* 转换流，同样要包装底层流（字节流对象）
  * 字符流和字节流可以相互转化
* `OutputStreamWriter`
  * 构造
    * `public OutputStreamWriter(OutPutStream out)`
    * `public OutputStreamWriter(OutPutStream out, String charsetName)`
      * charsetName 编码字符集
  * 普通
    * `public void write(int c)`
    * `public void write(char[] buffer)`
    * `public void write(char[] buffer, int offset, int len)`
    * `public void write(String str)`
    * `public void write(String str, int offset, int len)`
* `InputStreamReader`
  * 构造
    * `	public InputStreamReader(InputStream out)`		
    * `public InputStreamReader(InPutStream out, String charsetName)`
      * charsetName 解码字符集
  * 普通
    * `public void read()`
    * `public void read(char[] buffer)`
    * `public void read(char[] buffer, int offset, int len)`
* `flush()`
  * 调用编码器的 `flush()` 
  * 一定要 `flush()`
  * 当缓冲没有满的时候， `flush()` 刷新缓冲，输出到 底层流 传输数据
* `flush()` 与 `close() `
  * 调用 `close()` 前会自动调用 `flush()`，先将缓冲数据全部刷新到底层流，再进行传输
  * 传输完毕后，才关闭流，释放资源
* 复制
  * 复制文本文件很方便
  * 复制视频或文件等非文本文件会导致文件无法打开
    * 原因是 视频或图片没有按照字符集来编码的
    * 一些字符集没有的编码会被替换为默认码值，导致解码的时候无法完美还原

#### 转化流，装饰者设计模式

![转化流](http://on-img.com/chart_image/5a6ffdece4b024b99be485a4.png)

#### 简化字符流

* 名字太长，简化了书写，方法同上，方便
* 默认字符编码，默认缓冲大小，但是失去灵活性
* 直接传入文件对象或路径名，底层是字节流
* `FileWriter extends OutputStreamWriter `  写数据
  * `public FileWriter(String pathname)`
* `FileReader extends InputstreamReader `  读数据
  * `public FileWriter(String pathname)`

#### 字符缓冲流

* 引入原因：编码器有缓存，每次输出或输入到底层流的时候，会先放在缓存器的缓存中
  * 转化流每次与编码器交互的时候，每个字母编码器都会进行编码，一个一个字母编码影响效率
  * 在每次与编码器交互前，再提供一个缓存来存写和读的缓存


* 包装类，底层流
* `new BufferedReader(new InputStreamReader(new FileInputStream("test.txt")))`
* `new BufferedReader(new FileReader("test.txt"))`


* `BufferedWriter`
  * `public BufferedWriter(Writer out)`
  * `void newLine()`
* `BufferedReader`
  * `public BufferedReader(Reader in)`
  * `String readLine()`
    * 读取一行文本
    * 达到流末尾 返回 null
    * `while((s = br.readline()) != null) `
* `flush()`
  * 调用缓冲数组的 `flush()`
  * 缓冲满了才会通过 底层流 传输数据
  * 当缓冲没有满的时候， `flush()` 可以刷新缓冲，输出到 底层流 传输数据
* `flush()` 与 `close() `

#### 多级缓存写入

![多级缓存写入](http://on-img.com/chart_image/5a705098e4b024b99be5ea5c.png)

#### 打印流

* 只能输出
* `PrintStream` 字节打印流
  * `public PrintStream(File file)`  不用传入底层流，包装好了
    * 不带自动行刷新
  * `void print()`  
    * 可以打印任意类型的数据（boolean, int, byte, short, long, double, float)）
  * 不带自动刷新
* `PrintWriter` 字符打印流
  * `public PrintWriter(File file)`
    * 可以操作文件的流，或直接传入文件路径
    * 不带自动行刷新
  * `public PrintWriter(OutputStream out, boolean autoFlush)`
    * 开启自动行刷新功能
    *  `println()` `printf()` `format()`  只有这 3 个方法才可以自动  `flush()`
  * 有缓存，需要 `flush()` ，才能写入到底层流

#### 标准输入输出流

* System 里面 `in`  `out`  键盘 ／ 显示器
* `System.in`   类型是 `InputStream`
  * 调用时如果没有数据可读会处于阻塞等待输入
* `System.out`  类型是 `PrintStream` 
  * `PrintStream extends FilterOutputStream extends OutputStream`

#### 序列化流

* 将基本数据类型和引用数据类型写入到硬盘中
* 通过在流中使用文件（ 文件后缀 `.tmp` ），实现对象的持久化存储
* 对象要实现 `serializable` 接口，才可以被序列化
  * 类似于深拷贝 ，每个引用对象都要实现该接口，递归序列化引用变量
  * 即 Java 的序列化 就像一张对象网（graph）（图数据结构）
* `transient` 声明不需要序列化的成员变量


* `ObjectOutputStream ` 序列化
  * `public ObjectOutputStream(OutputStream out)`
  * `public void writeObject(Object o)`
* `ObjectInputStream`  反序列化
  * `public Object readObject()`
* 修改序列化后的序列化对象的类结构，反序列化会抛异常
  * `serializableVersionUID` 综合序列化类的各种信息（标示当前类的版本信息）	
  * 一旦改变，反序列化的版本信息不一致就会报错
  * 解决方法，自己定义类的 `private static final long serializableVersionUID`
* 应用
  * 用户退出当前应用，下次再打开，会恢复退出前状态
    * 将用户退出前状态作序列化存储，持久化存储
  * 远程调用：A 计算机创建的对象，在 B 计算机上执行
  * 可以实现深拷贝






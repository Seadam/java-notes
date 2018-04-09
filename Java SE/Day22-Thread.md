# Java Thread

#### 内容摘要

并发和并行，Thread API，线程优先级，线程生命周期，线程实现区别，线程安全问题，synchronized，同步，线程锁 ReentrantLock

------

#### 并发和并行

* 并发
  * 同一**时间段**执行多个任务（程序）
* 并行
  * 同一时刻（时间点）执行多个任务（程序）
* 区别
  * 在**单核**计算机中，一个 CPU **同一时刻**只能执行一个任务 —— 并发
  * **多核** CPU 同一时刻执行多个任务 —— 并行
* 原因
  * 在普通分时操作系统中（windows），以 10ms 为 CPU 运行时间片，虽然在同一时刻只能执行一个程序，但以 1秒 为单位来看的话，在这 1 秒 内有多个程序可以同时运行，即并发

#### *Thread API*

* `static void sleep(long mills);`

  * **当前**线程休眠（毫秒），线程进入阻塞状态
  * 休眠时间到后，进入就绪状态，等待 CPU
  * 在哪个线程中调用，哪个线程就会休眠
  * **推荐使用 `TimeUnit` 来休眠**
    * `TimeUnit.MILLISECONDS.sleep(3000)`  
    * `TimeUnit.SECONDS.sleep(3)`

* `public String getName()`

  * 返回当前线程的线程名

  * **拿到主线程的线程名**

    ```Java
    // Thread.currentThread 静态方法，拿到当前线程对象
    public static void main(String[] args) {
        System.out.println(Thread.currentThread.getName());
    }
    ```

* `public final void join()`

  * **当前调用线程 等待 当前执行线程执行完毕直至死亡**

  ```Java
  // 当前 main 线程调用 thread 线程 join
  // main 线程 等待 thread 线程执行完毕 才继续执行
  // 所以输出 0,1,2,3,4,5,6,7,8,9,0,1,2,3,4,5,6,7,8,9
  public static void main(String[] args) {
      Thread thread = new Thread(new Runnable() {
            for(int i = 0; i < 10; i++);
      });
      thread.join();
      for(int i = 0; i < 10; i++);
  }
  ```

* `public static void yield()`

  * 礼让，暂停当前调用线程，执行其他线程
  * 提示调度器当前线程可以礼让其他线程执行，但是调度器可以无视这个提示

  > A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint.

* `public final void setDaemon(boolean on)`

  * 守护线程，后台线程
  * 守护线程要在线程 `start()` 之前声明，即启动线程前调用该方法
  * 当正在运行的线程都是守护线程的时候，JVM 退出

* `public void interrupt()`

  * 打断处于 `wait()` `join()` `sleep()` 过程中的线程
  * 可以在一个线程中控制另一个线程的执行，**依赖于异常的代码控制**，即 `InterruptedException`
    * 抛异常原因
      * 处于等待状态或休眠状态的线程被中断，此时线程非正常终止
      * 从资源释放角度，比如某个线程在等待 I/O 时非正常终止 ，导致无法释放系统资源
      * 解决方法：在处理该异常的代码块中完成资源释放操作
  * 中断一个没有处于休眠或者等待状态的线程没有效果

#### 线程优先级 *priority*

* 范围 `1 ~ 10`
* `Thread.MAX_PRIORITY = 10` `Thread.MIN_PRIORITY = 1` `Thread.NORM_PRIORITY = 5`
* 设置优先级要在线程 `start()` 之前声明
* 同 `yield()` 类似，只是一个提示，不起决定作用，具体看调度器

#### 线程生命周期

![线程生命周期.png](http://upload-images.jianshu.io/upload_images/1904896-595f812436f3c2fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 线程实现区别

* 继承 Thread 类
  * 缺点：若某个类已经有父类，则无法再继承
* 实现 Runnable 接口
  * 解决单继承的局限性
  * **便于多线程间共享数据**
    * 继承类，声明多个线程的时候要创建多个对象，用 static 关键字进行共享变量
    * 实现接口，声明多个线程只需创建一个对象，调用多个 run 方法

```Java
// 共享数据：Runnable 接口实现清晰明了，Thread 继承实现也能做到
// 原理是：Thread 实现了 Runnable 接口
// 将继承 Thread 类对象当做接口传入 new Thread(Runnable runnable)里面，实现和接口同样功能

// 接口实现
public static void main(String[] args) {
  	Runnable r = new MyRunnableImplement();
  	new Thread(r).start();
    new Thread(r).start();
    new Thread(r).start();
  	// 三个线程对象持有同一个接口对象，run 方法相同，能够共享变量
}

// 继承实现
public static void main(String[] args) {
  	Thread t = new MyExtendsThread();
    new Thread(t).start();
    new Thread(t).start();
    new Thread(t).start();
  	// 三个线程对象持有同一个子类对象，并且父类实现了 Runnable 接口，可以当做参数传入，也能够共享变量
}

class MyRunnableImplement implements Runnable {
  	// 多个线程可以同时共享该变量
	int count = 100;
  	@Override
  	public void run() {
      	System.out.println(count--);
    }
}

class MyExtendsThread extends Thread {
  	// 多个线程可以同时共享该变量
	int count = 100;
  	@Override
  	public void run() {
      	System.out.println(count--);
    }
}
 
```

#### 线程安全问题

* 一旦涉及到多线程，线程之间的代码执行顺序**不确定性和偶现性**
* 线程安全问题
  * 数据共享
  * 多线程运行环境
  * 非原子操作
    * 原子操作：多个步骤之间不可分割
* 解决线程安全问题：
  * 非原子操作 ==> 原子操作
  * **在共享数据访问期间，阻止其他线程修改共享数据，则修改、访问操作加锁，执行完毕后自动释放锁**

#### *synchronized*

* 加锁，每一个对象中都有一个标志位来表示当前对象是否加锁
  * 锁：**是否有线程正在访问状态**，若有正在访问的线程，其他线程需要阻塞等待
  * 当前线程执行同步代码块代码时会先检查对象中是否有那把锁，若有锁，则等待持有锁的线程执行完毕释放锁后，当前线程才可以持有对象的锁继续执行同步代码块，若没有锁，拿到那把锁直接执行
* **对同一个共享数据只能使用同一把锁来解决线程安全问题**
* **锁可以是任意对象**
* **同步代码块：需要互斥访问的共享数据**（临界区）
  * 同步普通方法等价于 `synchronized(this)` 对象同步锁
  * 同步静态方法等价于 `synchronized(this.class)`  类同步锁

```Java 
// 每一个对象中都有一个标志位来表示当前对象是否加锁
// 所有对象都可以当锁
Object o = new Object();
synchronized (o) {
   // 同步代码块
}

// 同步方法，与同步代码块相比，同步方法不够灵活
public synchronized void sell() {
   // to do
}

// 隐式的指明同步的对象为 this ，等价于
public void sell() {
    synchronized(this) {
      // to do
    }
}
```

#### 同步

* 多个线程**使用同一个锁对象**，构成同步
* 解决了多线程安全问题
* `synchronized` 消耗资源，每次执行同步代码块前都要判断

#### 重入锁

* 加锁次数 + +
  * 同一把锁可以被一个线程多次持有
  * 同一个线程可以多次持有同一把锁


* `synchronized`  `ReentrantLock  `都是重入锁

```Java
synchronized (this) {
  	synchronized (this) {
  		synchronized (this) {
  	    	// 当前对象 加锁次数 3 次
		}	
	}
}
```

#### 线程锁 ReentrantLock

* `synchronized`
  * 加锁：在判断对象标志位的时候，JVM 加锁
  * 释放锁：当同步代码块中的代码执行完毕，JVM 为当前线程释放锁
* `Lock` 接口
  * `void lock()`
  * `void unlock()`
    * **必须要在 finally 释放锁**
* `ReentrantLock`
* 效率比 `synchronized` 高

```Java
Lock lock = new ReentrantLock();
lock.lock();
try {
   // 同步代码块
} finally {
  // 必须要在 finally 释放锁，JVM 不会帮忙释放
  lock.unlock(); 
}
```


# Java Thread

#### 内容摘要

多线程概述，继承 Thread 类，实现 Runnable 接口，Thread API，线程生命周期，启动匿名线程

---

#### 多线程概述

> * 多线程的执行路径
>   * 单线程：一条
>   * 多线程：多条
> * 进程：程序分配资源和运行的基本单位，正在操作系统上运行的程序就是进程
> * 进程上下文切换，资源消耗大，为了进一步提高 CPU 利用率，引入线程
> * 同一进程的各个线程之间共享资源，线程间切换代价小
> * 进程：资源分配的基本单位
> * 线程：程序执行的基本单位

* 线程调度策略：先来先服务(*FIFS*)，短作业优先(*SJF*)，分时调度(windows 10ms)，抢占式(*priority*)
* 每运行一个 Java 程序，都会相应启动一个 Java 虚拟机
* Java 中是使用抢占式，依靠优先级来调度线程

#### 实现方式一：继承 *Thread* 类

* 重写子类 `run()` 方法，创建线程
* `Thread` 类表示一个线程，**只有在 `run()` 方法体中执行的的代码**才会运行在 `Thread` 所代表的线程中
* 而 `Thread` 的**其他方法**运行在创建 `Thread` 的线程中（如 主线程创建该线程，则运行在主线程中）
* 一个线程只能启动一次，多次启动同一线程会出异常 `IllegalThreadStateException`

```Java
/* 
*  1. 创建 Thread 类的子类
*  2. 覆盖 run() 方法
*  3. 创建该类对象
*  4. 启动线程 start()
*     (thread.run() 只是方法调用，不能启动新线程)
*/
class MyThread extends Thread {
  	@Override
    public void run() {
		// 该方法体内执行的所有方法，包括方法调用，会运行在 Thread 所代表的线程中
    }
  
    public static void main(String[] args) {
		// 创建线程对象，并启动该线程
      	new MyThread().start();
    }
}
```

#### 实现方式二：实现 *Runnable* 接口

* 重写 `run()` 方法

```Java
/* 
*  1. 创建 Thread 类的子类
*  2. 重写 run() 方法
*  3. 创建该类对象
*  4. 启动线程 start()
*/
class MyThread implements Runnable {
  	@Override
    public void run() {
		// 该方法体内执行的所有方法，包括方法调用，会运行在 Thread 所代表的线程中
    } 
  	public static void main(String[] args) {
      	new Thread(new MyThread()).start();
    }
}
```

#### *Thread API*

* `static void sleep(long mills);`
* `public final void join()`
* `public static void yield()`
* `public final void setDeamon(boolean on)`
* `public void interrupt()`

#### 线程生命周期

![线程生命周期.png](http://upload-images.jianshu.io/upload_images/1904896-595f812436f3c2fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 启动匿名线程

```Java
// 方法一 匿名 Runnable 接口对象
new Thread(new Runnable() {
        @Override
        public void run() {
          System.out.println("hello");
        }}).start();

// 方法二 重写匿名 Thread 对象的 run 方法
new Thread() {
        @Override
        public void run() {
          System.out.println("world");
        }}.start();

// 形式三： 方法一 + 方法二 结合
// 输出的是 world 
/*
	Thread 原始 run 方法如下
	若传入重写了 run 方法的 Runnable 接口对象，则执行 target 的 run 方法
    @Override
    public void run() {
    	if (target != null) {
        	target.run();
        }
    }
    但此处 Thread 匿名类对象 方法覆盖 run 方法
    执行的是重写 Thread 匿名类对象的 run 方法
*/
new Thread(new Runnable() {
          @Override
          public void run() {
            System.out.println("hello");
          }
		}) {
          @Override
          public void run() {
            System.out.println("world");
          }
        }.start();
```








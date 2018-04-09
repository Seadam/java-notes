# Java Thread

#### 内容摘要

死锁，线程间通信，`wait()` 与 `sleep()` 区别，生产者消费者问题，配置文件 properties，多线程状态转换，线程池，定时器

------

#### 死锁

* 同步弊端
  * 效率问题
  * **嵌套锁**容易产生死锁问题
* 死锁问题
  * **两个以上的线程**在执行过程中，因争夺资源而产生**相互等待**的现象
* 开发中，锁或其他静态常量放在单独一个类
* 哲学家吃饭问题：5个哲学家，5支筷子

```Java
// 线程A 要先拿到 A锁，再拿到 B锁，才能执行
class A implements Runnable {
    @Override
    public void run() {
        synchronized (Lock.LOCK_A) {
            // sleep 才能看到死锁效果
            synchronized (Lock.LOCK_B) {
                System.out.println("class A print");
            }
        }
    }
}

// 线程B 要先拿到 B锁，再拿到 A锁，才能执行
class B implements Runnable {
    @Override
    public void run() {
        synchronized (Lock.LOCK_B) {
            // sleep 才能看到死锁效果
            synchronized (Lock.LOCK_A) {
                System.out.println("class B print");
            }
        }
    }
}

// 开发中，锁或其他静态常量放在单独一个类
public class Lock {
    public final static Object LOCK_A = new Object();
    public final static Object LOCK_B = new Object();
}
```

#### 线程间通信

* 针对同一个资源的操作有不同的线程来执行

* 等待唤醒机制  `Object`

  * `wait()`

    * 当前线程等待，直到其他线程调用 `notify()` or `notifyAll()`
    * **当前线程必须拥有锁对象，锁对象才能调用 `wait()` `notify()`**
    * **该线程放弃 CPU 执行权，释放锁，等待**
    * **直到其他线程（拥有该锁对象的线程）通过调用锁对象的 `notify()` or `notifyAll()`  来通知等待线程醒来**
    * 即**拥有同一把锁的多个线程间通信**

    > Causes the current thread to wait until another thread invokes the `notify()` method or the `notifyAll()`
    >
    > The current thread must own this object's monitor. The thread releases ownership of this monitor and waits until another thread notifies threads waiting on this object's monitor to wake up either through a call to the `notify` method or the `notifyAll` method

  * `notify()`

    * 唤醒在同一个锁对象上等待的一个线程
    * 直到当前线程释放此对象上的锁定（释放锁），才能继续执行被唤醒的线程
    * 简单来说，`notify()` 在代码中的位置，可以不一定在最后一行
    * 因为 `notify()` 仅唤醒了等待线程，当前线程若还没执行完（**锁还在**），等待线程就不会执行

  * `notifyAll()`

    * 唤醒全部等待线程，见上

```Java
// 仅使用一个类，实现2个线程交替打印 1 到 99
// 原理是，先 notify 再 wait
class Print implements Runnable {
    int i = 0;
    @Override
    public void run() {
        while (i < 100) {
            synchronized (this) {
                System.out.println(Thread.currentThread().getName() + " : " + i++);
                this.notify();
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
	public static void main(String[] args) {
        Runnable r = new Print();
        new Thread(r, "Thread 1").start();
        new Thread(r, "Thread 2").start();
    }
```

#### `wait()` 与 `sleep()` 区别

* 唤醒条件不同
  * `wait()` 需要拥有同一个锁对象的其他线程，通过调用锁对象的 `notify()` or `notifyAll()` 来唤醒
  * `sleep()` 睡眠时间到了，自己叫醒自己
* 对锁对象的操作不同
  * `wait()` 放弃 CPU 执行权，并释放锁
  * `sleep()` 放弃 CPU 执行权，不会释放锁

####生产者消费者模型

* 生产者 *producer*
  * 生产商品放入缓冲
  * 当缓冲满了等待消费者消费，通知消费者消费
* 消费者 *consumer*
  * 从缓冲中消费商品
  * 当缓冲为空等待生产者生产，通知生产者生产
* 缓存 *buffer*
* 代码，见day23.thread.edition1
* **注意事项**
  *  `synchronized` 锁对象应该是 **多线程共享的对象**
  * `wait()`，`notify()` 应该在 `synchronized` 代码块内
  * `wait()`，`notify()` 应该被  **多线程共享的对象 调用**
  * `wait()` 应该永远在 `while` 循环
    * 避免虚假唤醒：为防止该线程没有收到 `notify()` 调用也从 `wait()` 中返回
    * 线程被唤醒的前后都持续检查条件是否被满足

```Java
// 消费者生产者模型
synchronized (sharedObject) { 
    while (condition) { 
    	sharedObject.wait(); 
        // 释放锁，进入等待状态
    } 
    // 正常逻辑执行生产或者消费操作
}

// 也可以用 if／else
synchronized (sharedObject) { 
    if (condition) { 
    	sharedObject.wait(); 
        // 释放锁，进入等待状态
    } else {
        // 正常逻辑执行生产或者消费操作
    }
}
```

```Java
public class Test {
    public static void main(String[] args) {
        Container container = new Container();
        new Thread(new Producer(container), "Producer").start();
        new Thread(new Comsumer(container), "Consumer").start();
    }
}

// container
class Container {
    // JavaBean
    private Bean bean;
    // isExist
    private boolean isExist;

    // Producer put bean
    public synchronized void put(Bean bean) {
        // while bean is existent, Producer wait
        while (isExist) {
            try {
                this.wait();// try...catch
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // bean is not existent, Producer produce
        this.bean = bean;
        this.isExist = true;
        System.out.println(Thread.currentThread().getName() + " put -> " + bean);
        // notify Comsumer
        this.notifyAll();
    }

    // Comsumer get bean
    public synchronized Bean get() {
        // while bean is not existent, Comsumer wait
        while (!isExist) {
            try {
                this.wait();// try...catch
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // bean is existent, Comsumer comsume
        Bean bean = this.bean;
        this.bean = null;
        isExist = false;
        System.out.println(Thread.currentThread().getName() + " get -> " + bean);
        this.notifyAll();
        return bean;
    }
}

// Producer
class Producer implements Runnable {
    // Container
    Container container;

    public Producer(Container container) {
        this.container = container;
    }

    @Override
    public void run() {
        while (true) {
            container.put(new Bean("little bean"));
        }
    }
}

// Comsumer
class Comsumer implements Runnable {
    // Container
    Container container;

    public Comsumer(Container container) {
        this.container = container;
    }

    @Override
    public void run() {
        while (true) {
            Bean bean = container.get();
        }
    }
}

// JavaBean
class Bean {
    String name;
    public Bean(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Bean{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

#### 配置文件 *properties*

* 可持久化 

  * `store()`   stream

* 配置文件中每个键及其值都是一个**字符串**

  * 底层由 `HashTable` 实现，同步的，不允许 null key

  ```Java
  // 简单当做 键值都为 String 的 HashMap 来遍历
  // 读取配置文件方法
  		FileInputStream fis = null;
          Properties properties = new Properties();
          try {
              // 配置文件当前 project 的根目录下，可以使用相对路径
              fis = new FileInputStream("foods.properties");
              // 加载配置文件
              properties.load(fis);
          } catch (IOException e) {
              e.printStackTrace();
          }
          for (int i = 0; i < properties.size(); i++) {
              // 当做 key，value 都为 String 类型的 HashMap 来遍历
          }
  ```

####多线程状态转换

![线程状态图](http://on-img.com/chart_image/5a6b44fee4b0a92b46809f75.png)

#### 线程池

* 线程底层
  * 产生新的线程是操作系统的核心功能
  * Java 中通过 API 来产生新线程，底层由 JVM 通过操作系统内核提供的接口来产生新线程
  * JVM 要产生一个新线程必须与操作系统交互，交互需要效率代价
  * 从线程池拿线程省去交互时间提交效率


* 线程池里每一个线程代码执行后，线程不会死亡，而是回到线程池中成为空闲状态，等待下一次执行任务
* 工具类 `Executors` 静态方法创建线程池，返回 `ExecutorServise` 对象
* `public static ExecutorServise newCachedThreadPool()`
  * 创建一个缓存线程池，线程数量可以动态增长
  * 空闲线程超过 60 秒将会死亡
* `public static ExecutorServise newFixedThreadPool(int nThread)`
  * 创建一个固定线程数量的线程池，nThread 
* `public static ExecutorServise newSingleThreadPool()`
  * 创建一个只有一个线程的线程池，可以保证任务执行顺序
  * 将多个任务按先后顺序排序，执行

#### Executors

* 前提
  * `Callable`
    * 类似 `Runnable` ，但是能返回执行结果，若未返回任何值，会抛异常
  * `Future`
    * `Callable` 执行结果对象，是一个**未决结果对象**
    * `get()` 拿到执行结果，若线程还未返回，则将等待返回再拿到结果
    * 类似回调

* `public Future submit(Callable<V> task)`
  * 传入一个有返回结果的线程任务，返回一个表示未决结果的 Future
* `public Future submit(Runnable task)`
  * 传入一个无返回结果的线程任务，返回一个表示该任务的 Future
* `public void shutdown()`
  * 顺序关闭，执行以前提交的任务，不再接受新任务
* `public void shutdownNow()`
  * 停止所有正在执行的任务，暂停处在等待的任务，并返回等待任务列表

#### 定时器 Timer

* 线程工具，


* `Timer`
  * 调度器，调度 `TimerTask` 对象执行的时机和次数
  * 不是后台线程
  * `schedule(TimerTask task, long delay)`
    * 计划线程延迟 delay 毫秒启动
  * `schedule(TimerTask task, long delay, long period)`
    * 延迟 delay 毫秒，每 period 毫秒重复调度
  * `cancel()`
    * 停止调度器，并舍弃当前所有调度任务
* `TimerTask`
  * 计时器任务
  * `Timer` 调度的任务
  * `cancel()`
    * 取消此计时器任务
  * `run()`
    * 任务要执行的操作

```Java
 void schedule(TimerTask task, Date time) 
          // 安排在指定的时间执行指定的任务。 
 void schedule(TimerTask task, Date firstTime, long period) 
          // 安排指定的任务在指定的时间开始进行重复的固定延迟执行。 
 void schedule(TimerTask task, long delay) 
          // 安排在指定延迟后执行指定的任务。 
 void schedule(TimerTask task, long delay, long period) 
          // 安排指定的任务从指定的延迟后开始进行重复的固定延迟执行。 
 void scheduleAtFixedRate(TimerTask task, Date firstTime, long period) 
          // 安排指定的任务在指定的时间开始进行重复的固定速率执行。 
 void scheduleAtFixedRate(TimerTask task, long delay, long period) 
          // 安排指定的任务在指定的延迟后开始进行重复的固定速率执行。 
```
















# 表

#### 内容摘要

队列，树

---

#### 队列

> * 先进先出（FIFO）
> * 插入操作在队尾，删除操作在队首
> * 基本操作：`enqueue` 入队 `dequeue` 出队
> * 应用：排队队列，二叉树层次遍历

#### 链式映像

* extends myLinkedList 代码复用


* 入队 *enqueue*
  * 尾插法
* 查看对头元素 *poll*
  * 检查第一个元素存不存在
  * 返回第一个元素
* 出队 *dequeue*
  * remove(0)

#### 顺序映像（顺序循环队列）

* **Field**
  * front 队头指针
  * rear 队尾指针
  * 队空与队满判断：`rear == front` 条件不足，不能判断，解决方法：
    * 添加一个标志位，队满：`rear == front && flag == true`
    * 浪费一个空间，约定队头在队尾的下一个位置即为满 `(rear + 1) % elements.length == front`
* **Method**
  * enqueue
    * 检查容量
      * 扩容，注意需要多一个空间来判断是否队满，故要 + 2 `newCapacity(size + 2)`
      * 数组复制手写 `for(int i = front, j = 0; i != near; i = (i + 1) % length, j++)`
      * 赋值新指针
    * 入队
    * 移动指针 `rear = (rear + 1) % elements.length`
    * size ++
  * poll
    * 检查队列是否为空
    * 返回队头元素
  * dequeue
    * 拿到 poll 
    * 数据置空
    * 移动指针 `front =(front + 1) % elements.length`
    * size --

```Java
package day16.datastructure;
/*
        顺序队列简单实现
        队空： rear = front
        队满： (rear + 1) / length = front
        浪费一个空间，约定队头在队尾的下一个位置即为满
 */
public class MyQueue<T> {

    // 队头指针
    private int front;
    // 队尾指针
    private int rear;

    // 元素个数
    private int size;

    // 容器
    private Object[] elements;

    // 默认容量
    private final static int DEFAULT_CAPACITY = 10;

    // 最大容量
    private final static int MAX_QUEUE_CAPACITY = Integer.MAX_VALUE;

    public MyQueue() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public MyQueue(int capacity) {
        elements = new Object[capacity];
    }

    // 返回指针后移，循环，取模
    private int next(int i) {
        return (i + 1) % elements.length;
    }

    // 入队
    public void enqueue(T t) {
        // 判断容量是否足够入队
        ensureCapacity();
        // 入队
        elements[rear] = t;
        // 移动指针
        rear = next(rear);
        size++;
    }

    // 判断容量是否足够入队
    private void ensureCapacity() {
        // 判断是否队满 ：(rear + 1) / elements.length = front
        if (next(rear) == front) {
            grow();
        }

    }

    // 数组扩容
    private void grow() {
        // 注意：有个保留空间来确定队列是否满，+1
        //      新增空间，+1
        //      故 新数组为 size + 2
        Object[] temp = new Object[newCapacity(size + 2)];
        elements = arrayCopy(elements, front, temp, 0, size);

        // 赋值新指针
        front = 0;
        rear = size;
    }

    // 复制旧数组，返回新数组
    private Object[] arrayCopy(Object[] src, int srcPos, Object[] dest, int destPos, int length) {
        for (int i = front, j = destPos; i != rear; i = next(i), j++) {
            dest[j] = src[i];
        }
        return dest;
    }

    // 容量扩容
    private int newCapacity(int minCapacity) {
        int oldCapacity = elements.length;
        int newCapacity = oldCapacity;
        // 判断是否可以扩容，是否超过最大扩容
        if(minCapacity - oldCapacity > 0) {
            // 扩容
            newCapacity += (oldCapacity >> 1);

            // 临界问题
            if (newCapacity < 0) {
                newCapacity = hugeCapacity(minCapacity);
            }
        } else {
            throw new OutOfMemoryError();
        }
        return newCapacity;
    }

    // 临界问题
    private int hugeCapacity(int minCapacity) {
        if(minCapacity < 0) {
            throw new OutOfMemoryError();
        }
        return MAX_QUEUE_CAPACITY;
    }

    // 拿到队头元素
    public T poll() {
        // 判断队列是否为空
        ensureEmpty();
        T t = (T) elements[front];
        return t;
    }

    // 出队
    public T dequeue() {
        T t = poll();
        elements[front] = null;
        front = next(front);
        size--;
        return t;
    }

    private void ensureEmpty() {
        if(isEmpty()) {
            throw new IndexOutOfBoundsException("Queue is empty.");
        }
    }

    public boolean isEmpty() {
        return rear == front;
    }

    @Override
    public String toString() {
        if (isEmpty())
            return "[]";
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = front; i != rear; i = next(i)) {
            stringBuilder.append('[');
            stringBuilder.append(elements[i]);
            stringBuilder.append(']');
            if(next(i) == rear) break;
            stringBuilder.append("<- ");
        }
        return stringBuilder.toString();
    }
}
```

# 树

> * 节点
> * 度：节点的拥有子树的个数
> * 叶子（终端节点）：度为零的节点
> * 孩子：节点的子树的根
> * 分支节点：有孩子的节点
> * 兄弟：同一个双亲节点的子节点互称兄弟
> * 层次：根为第一层
> * 深度：树中节点的最大层次

#### 二叉树

* 每个节点最多有 2 颗子树，不存在度大于 2 的节点，有左右之分，有序
* **第 i 层**最多有 **(2^i-1^)** 个节点 ——每一层节点数
* **深度为 k 的二叉树**最多有 **(2^k^ - 1)** 个节点 ——节点总数
* 对于**任意一颗二叉树**，如果其**叶子（度为 0）节点树为 n0** ，**度为 2 的节点数为 n2** ，有 **n0 = n2 + 1**
  * 度为 0，1，2 的节点个数  n0， n1， n2
  * 二叉树所有节点个数：n0 + n1 + n2
  * 二叉树所有节点个数：n1 + 2 * n2 + 1
  * == > **n0 = n2 + 1**
* 具有 **n 个节点的完全二叉树**，其**最大深度为 log~2~n + 1**
  * 满二叉树：每一层的节点个数都为最多
  * 完全二叉树：将树从上到下，从左到右进行编号（从1开始），这棵树上每一个节点在**满二叉树**上对应
* 性质 5 堆的性质

#### 顺序映像

* 对于一般二叉树，可能会造成空间浪费很大，比如只有左子树，而右子树的空间一直空着
* 适合完全二叉树

#### 链式映像

> 线索二叉树
>
> * 2n 个父子节点
> * n - 1 个
> * n + 1 个空间会被浪费

* 遍历：访问数据结构中每一个元素，且只访问一次
  * 前序遍历：根结点 => 左子树 => 右子树
  * 中序遍历：左子树 => 根结点 => 右子树
  * 后序遍历：左子树 => 右子树 => 根结点
  * 层次遍历：每一层都从左向右遍历


















# 表

#### 内容摘要

链表，栈

------

#### 链式映像

> 用一组**任意**的存储单元存储线性表的数据元素
>
> * 由节点组成，节点包括数据和后继元素链，next 链，最后一个节点 next 链引用 null
> * 查找需要遍历整个链表，插入和删除开销小
> * 插入：新节点，再调整引用（ next 链），删除：直接调整引用即可

```java
// 插入
cur.next = pre.next;
pre.next = cur;

// 删除
pre.next = cur.next;

// 头插法：在第一个节点前插入，遍历序列是倒序
// 尾插法：在第一个节点后插入，遍历序列是顺序
```

##### `private static Node {}`

> **普通的成员内部类对象都隐式的持有外部类对象引用**
>
> * 当外部类对象没有引用指向它，即将成为垃圾时
> * 内部类对象还隐式指向它，导致无法回收，可能造成内存泄漏
> * 若内部类加了 static 修饰，那么内部类对象不再持有外部类对象的引用，可以规避

* 添加节点 *add*
  * 检查下标合法性 [0, size]
  * 找前节点 (第一个节点的前节点为 null)
  * 链接
* 链接 *link*
  * 判断前节点是否为空，即在第一个节点前插入
  * 若尾指针为空，指向当前插入节点
  * 若最后一个插入，移动尾指针
  * size ++
* 删除节点 *remove*
  * 检查下标合法性 [0, size - 1]
  * 找前节点 (第一个节点的前节点为 null)
  * 断链
  * 返回删除节点值
* 断链 *unlink*
  * 判断前节点是否为空，即删除第一个节点
  * 若第一个节点为空，即 size = 0，尾指针置空
  * 若删除最后一个节点，移动尾指针
  * size --

```Java 
package day15.datastructure;

/*
    单链表，无头节点，简单实现 LinkedList
    泛型 T
 */
public class MyLinkedList<T> {

    // 指向第一个节点指针
    private Node<T> first;

    // 指向尾节点，方便操作
    private Node<T> last;

    private int size;


    // 头插法添加节点
    public void addHead(T t) {
        link(t, null);
    }

    // 尾插法添加节点
    public void addTail(T t) {
        link(t, last);
    }

    // 根据下标添加节点
    public void add(T t, int index) {
        // 检查下标合法性
        rangeCheckForAdd(index);

        // 拿到当前节点前节点
        Node pre = preNode(index);

        // 链接
        link(t, pre);
    }

    // 遍历出当前下标之前节点
    private Node<T> preNode(int index) {
        // 若是要找第一个节点的前节点
        if (null == first || index == 0) {
            return null;
        }

        // 从第一个节点开始
        Node node = first;

        for (int i = 1; i < index; i++) {
            node = node.next;
        }
        return node;
    }

    // 链接操作，处理各种情况
    private void link(T t, Node pre) {
        // 若前节点是空节点，即在第一个节点前链接
        if (null == pre) {
            Node<T> node = new Node(t, first);
            first = node;
            if (null == last) {
                last = first;
            }
        } else {
            // 正常链接
            Node cur = new Node(t, pre.next);
            pre.next = cur;

            // 若此时插入的是最后一个节点，调整 尾指针
            if (cur.next == null) {
                last = cur;
            }
        }

        size++;
    }

    // 检查添加下标合法性
    private void rangeCheckForAdd(int index) {
        if (!isPositionIndex(index))
            throw new ArrayIndexOutOfBoundsException("index out of bounds: index: " + index + " size: " + size);

    }

    // 是否在添加操作合法下标内
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    // 删除下标对应节点
    public T remove(int index) {
        // 判断下标边界
        rangeCheckForRemove(index);

        // 拿到要删除节点前一个节点
        Node<T> pre = preNode(index);

        // 断开链接
        Node<T> cur = unlink(pre);
        return cur.item;
    }

    // 删除第一个节点
    public T removeHead() {
        return unlink(null).item;
    }

    // 删除最后一个节点
    public T removeTail() {
        return unlink(preNode(size - 1)).item;
    }

    // 断链操作，处理各种情况
    private Node<T> unlink(Node<T> pre) {
        Node<T> cur = first;
        // 若删除第一个节点，即第一个前节点为空
        if (null == pre) {
            first = first.next;
            // 若第一个节点为空，即 size = 0，尾指针置空
            if(null == first) {
                last = null;
            }
        } else {
            // 正常断链
            cur = pre.next;
            pre.next = cur.next;

            // 若此时删除的是最后一个节点
            if (cur.next == null) {
                last = pre;
            }
        }

        size--;
        return cur;
    }

    // 根据下标值，拿到节点内容
    public T valueOf(int index) {
        rangeCheckForRemove(index);
        Node<T> node = preNode(index);
        return node.next.item;
    }

    // 检查添加下标合法性
    private void rangeCheckForRemove(int index) {
        if (!(isElementIndex(index)))
            throw new ArrayIndexOutOfBoundsException("index out of bounds: index: " + index + " size: " + size);

    }

    // 是否在删除操作合法下标内
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    @Override
    public String toString() {
        if (isEmpty())
            return "null";
        StringBuilder stringBuilder = new StringBuilder();
        for (Node node = first; ; node = node.next) {
            stringBuilder.append('[');
            stringBuilder.append(node.item);
            stringBuilder.append(']');
            if (node.next == null) {
                break;
            }
            stringBuilder.append("-> ");
        }
        stringBuilder.append("-> null");
        return stringBuilder.toString();
    }

    // 能避免内存泄漏问题，详细看笔记
    // 私有静态成员内部类，节点
    private static class Node<T> {

        T item;
        Node<T> next;

        public Node(T item, Node next) {
            this.item = item;
            this.next = next;
        }
    }
}

```

#### 栈

> * 后进先出（LIFO）
> * 插入和删除操作只能在栈顶执行
> * 基本操作：`push` 进栈 `pop` 出栈
> * 应用：符号匹配，表达式求值，方法调用

* push
  * 检查容量
  * 扩容
  * 压入栈中，指针上移
* pop
  * 检查栈是否为空
  * 出栈，指针下移
* peek
  * 检查栈是否为空
  * 访问

```Java
package day15.datastructure;

import java.util.Arrays;

/*
        顺序栈简单实现
        泛型 T
        借助 Arrays 复制数组
 */
public class MyStack<T> {

    // 栈顶指针，也可以表示栈内元素个数
    private int top;

    // 存储数据元素
    private Object[] stack;

    // 默认栈容量
    private static final int DEFAULT_CAPACITY = 10;

    // 最大栈容量
    private static final int MAX_STACK_SIZE = Integer.MAX_VALUE;

    public MyStack() {
        stack = new Object[DEFAULT_CAPACITY];
    }

    // 出栈
    public T pop() {
        rangeCheck();
        return (T)stack[--top];
    }

    // 入栈
    public void push(T t) {
        // 检查容量合法性
        ensureCapacity();

        stack[top++] = t;
    }

    // 访问栈顶元素
    public T peek() {
        rangeCheck();
        return (T)stack[top];
    }

    public int size() {
        return top;
    }

    public boolean isEmpty() {
        return top == 0;
    }

    // 检查栈是否为空
    private void rangeCheck() {
        if (isEmpty())
            throw new ArrayIndexOutOfBoundsException("index out of bounds top: " + top + " size: " + top);
    }

    // 检查栈容量是否需要扩容
    private void ensureCapacity() {
        if (top == stack.length)
            grow();
    }

    // 数组扩容
    private void grow() {
       stack = Arrays.copyOf(stack, newCapacity(top + 1), Object[].class);
    }

    private int newCapacity(int minCapacity) {
        int oldCapacity = top;
        int newCapacity = 0;
        // 当发生越界的时候
        // minCapacity = Integer.MAX_VALUE + 1 - Integer.MAXVALUE
        if (minCapacity - stack.length > 0) {

            // 扩容，1.5 倍
            newCapacity = oldCapacity + (oldCapacity >> 1);

            // 临界问题
            if(newCapacity < 0) {
                newCapacity = hugeCapacity(minCapacity);
            }
        } else {
            throw new OutOfMemoryError();
        }
        return newCapacity;

    }

    // 临界问题，直接扩容至最大容量
    // 扩存放元素还未达到上限，但扩容后容量越界，则直接将容量设置为最大容量
    private int hugeCapacity(int minCapacity) {
        if(minCapacity < 0)
            throw new OutOfMemoryError();
        return MAX_STACK_SIZE;
    }

    @Override
    public String toString() {
        if(isEmpty()) {
            return "[]";
        }

        StringBuilder stringBuilder = new StringBuilder();
        for (int i = top - 1; i >= 0; i--) {
            stringBuilder.append("\n[");
            stringBuilder.append(stack[i]);
            stringBuilder.append(']');
        }
        stringBuilder.append("\n[_]");
        return stringBuilder.toString();
    }
}

```

#### 队列

> * 先进先出（FIFO）
> * 插入操作在队尾，删除操作在队首
> * 基本操作：`enqueue` 入队 `dequeue` 出队
> * 应用：排队队列，二叉树层次遍历
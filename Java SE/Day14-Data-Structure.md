# 数据结构

#### 内容摘要

数据结构，线性表

------

#### 数据结构

* 相互之间存在一种或多种特定关系的数据元素的结合
  * 数据元素：数据
  * 关系（逻辑表示）：
    * 集合：所以数据元素同属于一个集合，数据元素之间没有其他任何关系
    * 线性：数据元素之间存在一一对应的关系，前驱和后继
    * 树：一对多的关系
    * 图：多对多的关系
  * 物理结构：数据结构在计算机中的表示
* 物理结构
  * 顺序映像
    * 元素之间物理位置紧邻
    * 一片连续的内存空间
  * 链式映像
    * 单链表，双向链表，循环链表，静态链表
    * 元素之间不是物理位置紧邻
    * 不仅存储数据元素，还存储指向下一个数据元素的引用

# 表

> 一个数据元素可以由**有限个**数据项组成。数据元素称为记录，含有大量记录的线性表又称为文件。这种结构具有下列特点：存在一个唯一的没有前驱的（头）数据元素；存在一个唯一的没有后继的（尾）数据元素；此外，每一个数据元素均有一个直接前驱和一个直接后继数据元素。

#### 顺序映像

> 用一组**地址连续**的存储单元依次存储线性表的数据元素
>
> * 随机存取
> * 查找不费时间，插入和删除开销大
> * 插入：后续元素需要后移，删除：后续元素需要前移

* 注意事项：
  * 元素后移，元素前移
  * 扩容问题
  * 边界检查
  * null 检查
* Add
  * 位序是否合法 [0, size]
  * 检查容量，当前数组容量已被装满
    * 扩容，是否超过容量上限
    * 临界问题
    * 复制新数组
  * 元素后移
  * 插入新元素
  * size++
* Remove
  * 位序是否合法 [0, size - 1]
  * 删除该元素
  * 元素前移
  * size--

```Java
package day14.datastructure;
/*
    简单实现Arraylist
    借助System.arraycopy
        Arrays.copyOf
 */

import java.util.Arrays;

// 泛型 T
// Object[] 数组
public class MyArrayList<T> {

    // 数组存放数据元素
    private Object[] elements;

    // 记录当前元素个数
    private int size;

    // 最大数组容量
    private final static int MAX_ARRAY_SIZE = Integer.MAX_VALUE;

    // 默认初始化容量
    private final static int DEFAULT_CAPACITY = 10;

    // 无参构造方法，构造一个默认容量的数组
    public MyArrayList() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    // 构造一个指定容量大小的数组
    public MyArrayList(int capacity) {
        // 判断容量合法性
        if (capacity < 0) {
            throw new IllegalArgumentException("illegal argument " + capacity);
        }

        elements = new Object[capacity];
    }

    // 直接添加元素
    public void add(T t) {
        ensureCapacity();
        elements[size] = t;
        size++;
    }


    // 按下标添加元素
    public void add(T t, int index) {
        // 检查下标合法性
        checkForAddIndex(index);

        // 检查数组容量，扩容
        ensureCapacity();

        // 数组元素后移
        moveBack(index);
//        循环后移
//        for (int i = size; i > index; i--) {
//            elements[i] = elements[i - 1];
//        }

        // 插入操作
        elements[index] = t;

        // 元素个数加加
        size++;
    }

    // 数组元素后移
    private void moveBack(int index) {
        // (size - 1) 最后一个元素下标
        // (size - 1) - index + 1 需要移动的距离
        int moveLength = size - index;

        // 数组复制
        // index ：从当前下标开始复制
        // index + 1 ：复制到当前下标后一项
        // 即从当前下标整个后移一项
        System.arraycopy(elements, index, elements, index + 1, moveLength);
    }

    // 检查插入下标合法性，[0, size]
    private void checkForAddIndex(int index) {
        if (index < 0 || index > size)
            throw new ArrayIndexOutOfBoundsException("index out of bounds: index: " + index + " size: " + size);

    }

    // 检查数组容量
    private void ensureCapacity() {
        if (size == elements.length)
            newCapacity(size + 1);
    }

    // 扩容
    private void newCapacity(int minCapacity) {

        // 先确定是否达到最大容量
        // 若 elements.length 还没达到最大容量，则可以扩容
        // minCapacity = Integer.MAX_VALUE + 1 - Integer.MAX_VALUE = 1
        if (minCapacity - elements.length > 0) {

            // 扩容， 扩容倍率为 1.5 ,采用移位操作
            int oldCapacity = elements.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);

            // 临界问题
            // 扩容后容量达到上限，值超过最大整数即为负，但数组中还有元素可以插入，则将容量设为最大容量
            if (newCapacity < 0)
                newCapacity = hugeCapacity(minCapacity);
            // 开始扩容，即复制一个新数组
            elements = Arrays.copyOf(elements, newCapacity, Object[].class);
        } else {
            throw new OutOfMemoryError();
        }

    }

    // 扩容后容量达到上限，值超过最大整数即为负，但数组中还有元素可以插入，则将容量设为最大容量
    private int hugeCapacity(int minCapacity) {
        if(minCapacity < 0)
            throw new OutOfMemoryError();
        return MAX_ARRAY_SIZE;
    }

    // 检查删除下标合法性，[0, size - 1]
    private void checkForGetIndex(int index) {
        if (index < 0 || index >= size)
            throw new ArrayIndexOutOfBoundsException("index out of bounds: index: " + index + " size: " + size);

    }

    // 数组元素前移
    private void moveForward(int index) {
        // (size - 1) 最后一个元素下标
        // (size - 1) - index + 1 - 1 需要移动的距离
        int moveLength = size - index - 1;

        if(moveLength > 0)
        // 数组复制
        // index + 1 ：从当前下标后一项开始复制
        // index ：复制到当前下标
        // 即从当前下标后一项整个前移一项
            System.arraycopy(elements, index + 1, elements, index, moveLength);
    }

    // 删除元素
    public T remove(int index) {
        // 检查下标合法性
        checkForGetIndex(index);

        // 拿到待删除元素
        T t = (T) elements[index];

        // 数组元素前移
        moveForward(index);
//        循环前移
//        for (int i = index; i < size - 1; i++) {
//            elements[i] = elements[i + 1];
//        }

        // 最后一个元素赋值为空
        elements[size - 1] = null;

        // 元素个数减减
        size--;
        return t;
    }

    // 根据值，找到对应下标
    public int indexOf(T t) {
        // 若该值为 null 则使用 == 比较
        if (null == t) {
            for (int i = 0; i < size; i++) {
                if(null == elements[i])
                    return i;
            }
        }

        // 若不为空，则用 equals 比较
        for (int i = 0; i < size; i++) {
            if(t.equals(elements[i]))
                return i;
        }
        return -1;
    }

    // 拿到当前下标值元素
    public T get(int index) {
        checkForGetIndex(index);
        return (T)elements[index];
    }

    public boolean isEmpty() {
        return size == 0;
    }

    @Override
    public String toString() {
        if (isEmpty())
            return "[]";
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i < size - 1; i++) {
            sb.append(elements[i]);
            sb.append(", ");
        }
        sb.append(elements[size - 1]).append(']');
        return sb.toString();
    }
}

```

#### 链式映像

> 用一组**任意**的存储单元存储线性表的数据元素
>
> * 由节点组成，节点包括数据和后继元素链，next 链，最后一个节点 next 链引用 null
> * 查找需要遍历整个链表，插入和删除开销小
> * 插入：新节点，再调整引用（ next 链），删除：直接调整引用即可

```Java
// 插入
cur.next = pre.next;
pre.next = cur;

// 删除
pre.next = cur.next;
```

##### `private static Node {}`

> **普通的成员内部类对象都隐式的持有外部类对象引用**
>
> * 当外部类对象没有引用指向它，即将成为垃圾时
> * 内部类对象还隐式指向它，导致无法回收，可能造成内存泄漏
> * 若内部类加了 static 修饰，那么内部类对象不再持有外部类对象的引用，可以规避

# 数组动态扩容

* 顺序表，顺序栈，关于数组动态扩容边界检查问题
  1. `ensureCapacity` 先检查是否需要扩容
  2. `grow` 执行数组复制操作
  3. `newCapacity` 扩容，返回新扩容后容量
  4. `hugeCapacity` 是否需要扩容至最大容量判断

```Java
// 检查栈容量是否需要扩容
    private void ensureCapacity(int index) {
        if (index == elements.length)
            grow();
    }
    // 数组扩容
    private void grow() {
       elements = Arrays.copyOf(elements, newCapacity(size + 1), Object[].class);
    }
    private int newCapacity(int minCapacity) {
        int oldCapacity = elements.length;
        int newCapacity = oldCapacity;
        // 当发生越界的时候
        // minCapacity = Integer.MAX_VALUE + 1 - Integer.MAXVALUE
        if (minCapacity - oldCapacity > 0) {

            // 扩容，1.5 倍
          	// newCapacity += (oldCapacity >> 1)
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
        return MAX_ARRAY_SIZE;
    }

```


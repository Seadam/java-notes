# Java API Collection

#### 内容摘要

Collection，Iterator，List，ArrayList，增强形式 for 循环，Set，HashSet

---

* 集合 与 数组
  * 集合元素个数可变，只能存储对象类型，有包装类，自动装箱和拆箱
  * 数组元素个数不可变，可以存储基本数据类型，也可以存储对象类型

#### *Collection*

当集合中没有使用泛型时，默认存储元素为 Object 类型 (*raw type* 原始类型) (*primitive type* 基础类型)

`Collection` <interface> 对于多个对象的所有基本操作

* `List` <interface>
  * 元素有序，可重复
  * `ArrayList`	
  * `LinkedList`
  * `Vector` 可增长数组（线程安全）

* `Set` <interface>
  * 元素无序，且唯一（元素插入顺序与取出顺序不一致）
  * `HashSet` 哈希码
  * `TreeSet` 红黑树
* API  (*Application Program Interface*)
  * add() 添加元素
  * remove(Object o) 返回第一次找到 o 元素的下标
  * contains(Object o) 是否有 o 这个元素
  * clear() 清空当前集合中的元素
  * addAll(Collection c) 
  * removeAll(Collection c) 根据 c 中元素，删除当前集合中所有相同元素
  * containsAll(Collection c) 判断两个集合是否相交
  * retainAll(Collection c) 返回当前集合中元素求交集后是否发生改变，求当前集合与 c 的交集
* 遍历
  * Object[] toArray(); 返回 Object[] 数组再遍历
  * Iterator iterator(); 返回迭代器再进行迭代

#### *Iterator* <interface>

* 对集合中的元素进行遍历的迭代器
* 依赖于当前集合中元素
* 游标初始位置在第一个元素之前
* 遍历完成后，游标位于最后一个元素
* **游标遍历时，位于两个元素之间**
* Iterator 设计模式

```Java
boolean hasNext(); // 下一个元素是否存在
E next(); // 拿到当前游标元素
boolean remove(); // 删除当前游标元素
```

#### *List*

* 位序 `index` ，可以对元素插入位置精确控制
* 允许元素重复
* ListIterator listIterator(); 

#### *ListerIterator*

* 添加，删除，向前遍历，向后遍历
* 其他见上
* 可以逆序遍历

```Java
boolean hasPrevious(); // 上一个元素是否存在
E previous(); // 拿到当前游标元素，可以逆序遍历
void add(E e); // 添加元素
void set(E e);
int nextIndex();
int previousIndex();

// ConcurrentModificationException
```

* `ConcurrentModificationException`
  * 在迭代器遍历过程中，调用**集合的添加或删除方法**修改集合中的元素，如果继续迭代，会出现上述异常
  * 由于元素个数发生改变，如果要继续使用迭代器来遍历，需要修改后集合获得的新迭代器
* 解决办法
  1. **迭代器遍历，迭代器操作**，即不用集合中添加方法，而用**迭代器的 `add()` `remove()`方法**
  2. 集合遍历，集合添加

#### *ArrayList*

* 底层是数组
* 查询快
* 增删慢，移动元素
* 线程不安全

#### 增强形式 for 循环

* 要判断需要遍历的目标是否为 null
* `for ( 数据类型 变量名 ： 集合或数组)`
* `foreach` 语句
* `iter` IDEA 快捷

```Java
for(String s:strings) {
  System.out.println(s);
}
// s 当前访问的对象
// String 访问对象的类型
// strings 需要遍历的数组或集合
```

#### *Set*

* 元素不可重复
* 元素**无序**：元素插入顺序与取出顺序不一致
* 底层由 `HashMap` 实现
  * map：健值映射 `key => value`  
  * `HashMap` 中 **`key` 唯一**
  * **将 `HashSet` 中元素 当做 `key` 传入 `HashMap` 中实现元素不可重复**

#### *HashSet*

* 实现 key 唯一的方法即：哈希值相同且对象内容相同


* 对于自定义类的不同的对象类型，若要实现插入的值唯一，需要要重写 `equals()`  `hashCode()` 方法
* 因为调用其默认的 `euqals()` 方法时，比较的是引用地址值，地址值不同 key 也不同
* 并且默认的 `hashCode()` 方法，采用的是引用地址值进行 hash，地址值不同哈希值也不同
* 故无法实现 key 唯一，所以要重写

#### *TreeSet*

* 见 Day20














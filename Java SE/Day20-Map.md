# Java API Map 

#### 内容摘要

HashMap，TreeSet，泛型，Comparable，Comparator，Map，嵌套，多线程概述

------

#### *HashMap*

* `table` 哈希表 数组 - 链表
* `key - value` 键值对


* 冲突域：`hash` 值相同并且 `key` 值相同
* 遍历顺序跟插入顺序不同，其实是按哈希表顺序遍历
* 判断 `key` 值重复，保证键值的唯一性：
  1. `hash` 值相同，`key` 对象是同一对象 ( `==` )
  2. `hash` 值相同，`key` 对象的内容相同 ( `equals` )
  3. `p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))`

* 1.7 HashMap jdk
  * `key` 值为 `null` 的节点，放在 `table[0]` 的链表中
*  1.8 HashMap jdk
  * `hash = k.hashCode() ^ (k.hashCode() >>> 16)`
  * `index = hash & (n - 1)`  （ n = table.length ）
  * `key` 值为 `null` 的节点，放在 `table[0]` 的链表中
  * **链表节点超过 8 个后，转化为红黑树**  `TREEIFY_THRESHOLD = 8`

```Java
// jdk 1.7 HashMap put
   
   public V put(K key, V value) {
     	// 若 key 值为 null，把值放入 null key 对应的 table 中
        if (key == null)
            return putForNullKey(value);
		// 针对 key 值的 hashCode，计算出其 hash 值
        int hash = hash(key.hashCode());
		// 针对计算出的 hash 值，计算出其对应的 table 中的下标
        int i = indexFor(hash, table.length);
		
     	// 1. 计算出下标，就到 table 中取出相应链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
           // 2. 遍历该链表，比较每一个元素节点中的元素的 hash 值以及 key 值的内容
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
              	// 3. 当且仅当待插入元素与当前访问中的链表中的元素，其hash值与key的内容都相同的时候
				//    返回该已经存在的 key 值所对应的 value 值，新值覆盖旧值
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
     	// 4. 如果遍历完该链表，没有元素的 key 与待插入的 key 相同,于是插入这个 key
        addEntry(hash, key, value, i);
        return null;
    }

	// 把值放入 null key 对应的 table 中
    private V putForNullKey(V value) {
		// 在 table 中 null key 所对应的链表固定存储在 table 的第 0 个单元
		
		// 1. 遍历hash表中第0个单元所对应的链表
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
         	// 2. 对于链表当中的每一个元素，比较器 key 值
			// 3. 当存在 key 值为 null 的元素，就覆盖该链表节点中的 value
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
      	// 4. 如果当遍历完整个链表，发现没有 key 为 null 的元素节点存在
		//    此时，才新建一个 key 值为 null 的节点，插入到该链表中
        addEntry(0, null, value, 0);
        return null;
    }
	
	// 添加一个新的节点
	void addEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }

```

```Java
// jdk 1.8 HashMap putVal

	// 红黑树化 阈值
	static final int TREEIFY_THRESHOLD = 8;

	final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
      	// 如果 table 为空，则进行初始化空间操作
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
      	// 若 key = null，hash = 0；==> null key 存储在 table 的第 0 个单元
      	// 若 table[0] == null，说明链表里还没有元素，则插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {// 否则，链表里还有元素
            Node<K,V> e; K k;
            // 若链表的第一个元素与当前插入 hash 值和 key 值相同的话，则拿到第一个元素
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 若链表的元素是树节点之后，执行插入树节点操作，并且拿到返回的树节点
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { // 若链表的元素不是树节点
              	// 计数遍历链表，插入到链表尾
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                      	// 若当前链表节点超过 8 个，执行树化操作，将链表转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  	// 若链表元素没有超过 8 个
                    // 并且找到与当前插入 hash 值和 key 值相同的元素，拿到当前元素
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
          	// 以上拿到的元素不为 null 时，旧值覆盖新值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
              	// 钩子方法，没有方法体，为开发者提供接口
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
      	// 若添加新 key 后的容量大于阈值后，扩容
        if (++size > threshold)
            resize();
      	// 钩子方法，没有方法体，为开发者提供接口
        afterNodeInsertion(evict);
        return null;
    }
```

#### *TreeSet*

* 底层是由红黑树实现
* 插入无序，遍历自然顺序，或自定义比较规则来排序
  1. 比较对象实现 `Comparable` 接口
  2. 向 `TreeSet`  传入比较器，指明 `Comparator` 排序规则
* **当按自然顺序添加完元素后，修改某一元素，不会对元素重新排序**
  * **解决方法：将其删除再添加**
* 元素唯一，元素不重复，为了确保 key 值唯一
  * 主要看 `compareTo` 的返回值来判断 `key` 是否相等
  * `x.compareTo(y) == 0` 必须与 `x.equals(y)` 一致
* `add` `remove` `contains` 时间复杂度log(n)
* 自定义对象无法直接放入 `TreeSet` 中，因为没有指明自定义对象的比较规则，无法进行比较
  * `ClassCastException`
  * 解决方法：
    * 直接将比较规则 `Comparator` 告诉 `TreeSet`
    * 自定义对象实现接口 `Comparable` ，实现自己的比较规则  

#### 泛型

* 指明集合类中放入元素的类型
* 若不用泛型，则可以放入任何 Object 类型的数据，导致使用不便

#### *Comparable* 接口

* `(x.compareTo(y) == 0) == (x.equals(y));`
* `compareTo(Object o)`
  * 比较值：负整数，零，正整数
  * 对应比较结果：小于，等于，大于


* **可以使用泛型**

```Java
class Student implements Comparable<Student> {
    @Override
    public int compareTo(Student o) {
      // 自定义比较规则
      // 正整数 : >
      // 零    : ==
      // 负整数 : <
    }
}
```

#### *Comparator* 接口

* 可以使用泛型
* 可以用匿名类

```Java
TreeSet<Student> treeSet = new TreeSet<>(new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
      // 自定义比较规则
      // 同上
    }
});
```

#### *Map*

* `key - value` 健值对

* API
  * V put(K key, V value);
  * V remove(Object key);
  * void clear();
  * boolean containsKey(K key);
  * boolean containsValue(V value);
  * boolean isEmpty();
  * int size();
  * V get(Object key);
  * Set<K> keySet();
    * key 值不能重复，返回的是 set 集合
    * 可以通过 Set 遍历整个 Map 键值对
  * Collection<V> values();
  * Set<Map.Entry<K, V>> entrySet();
    * 键值对 Set 集合，可以遍历整个 Map
* 遍历
  * 通过 `Set<K> keySet();` 拿到 Set 集合遍历键值对
  * 通过 `Set<Map.Entry<K, V>> entrySet()` 拿到键值对集合遍历
  * 遍历顺序跟插入顺序不同，其实是按哈希表顺序遍历

#### 嵌套

```java
// Collection<Collection<Integer>> 集合可以嵌套
// HashMap<Ingeter, HashMap<>> Map 可以嵌套
```

#### 多线程概述

* 多线程的执行路径
  * 单线程：一条
  * 多线程：多条
* 进程：程序分配资源和运行的基本单位，正在操作系统上运行的程序就是进程
* 进程上下文切换，资源消耗大，为了进一步提高 CPU 利用率，引入线程
* 同一进程的各个线程之间共享资源，线程间切换代价小
* 进程：资源分配的基本单位
* 线程：程序执行的基本单位
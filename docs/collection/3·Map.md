# 3·Map

- [3·Map](#3map)
  - [Map是什么](#map是什么)
  - [HashMap](#hashmap)
    - [重要参数](#重要参数)
    - [put方法的流程](#put方法的流程)
  - [HashTable](#hashtable)
  - [TreeMap](#treemap)
  - [LinkedHashMap](#linkedhashmap)
  - [ConCurrentHashMap](#concurrenthashmap)

## Map是什么
Map集合接口以键值对方式（key-value）存储数据。

特性：
1. Map集合不能存放重复元素(key不能重复) 。
2.  如果key重复时覆盖旧数据。

## HashMap
HashMap是Map接口的具体实现类，基于哈希表实现Map接口，以key-value键值对形式存储数据，
JDK1.7 底层数据结构使用的是数组 + 链表。
JDK1.8 底层数据结构使用的是数组 + 链表 / 红黑树。

特性：
- 无序排列。
- 非线程安全。在多线程的使用场景中，尽量使用线程安全的ConcurrentHashMap。Hashtable效率太低。
- 允许存放null键和null值。HashMap中只能有一个key为null的节点。因为Map的key相同时，后面的节点会替换之前相同key的节点。

jdk1.7中使用数组加链表的方式，在进行链表插入时候使用的是头结点插入的方法。不用每次进行遍历链表的长度。但是这样会有一个缺点，在进行扩容时候，会导致进入新数组时候出现倒序的情况，也会在多线程时候出现线程的不安全性。
jdk1.8中总是要进行阙值的判断，判断在什么时候进行红黑树和链表的转换。所以无论什么时候都要进行遍历，于是插入到尾部，防止出现扩容时候还会出现倒序情况。避免了线程死锁的问题。

### 重要参数
- `DEFAULT_INITIAL_CAPACITY`：Table数组的初始化长度`1 << 4`.
- `MAXIMUM_CAPACITY`：Table数组的最大长度`1 << 30`.
- `DEFAULT_LOAD_FACTOR`：负载因子，默认值为0.75。当元素的总个数 > 当前数组的长度 * 负载因子。数组会进行扩容，扩容为原来的两倍。
- `TREEIFY_THRESHOLD`：链表树化阙值，默认值为 8 。表示在一个node（Table）节点下的值的个数大于8时候，会将链表转换成为红黑树。
- `UNTREEIFY_THRESHOLD`：红黑树链化阙值，默认值为 6。表示在进行扩容期间，单个Node节点下的红黑树节点的个数小于6时候，会将红黑树转化成为链表。
- `MIN_TREEIFY_CAPACITY`：最小树化阈值，64，当Table数组元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）。

### put方法的流程
1. 首先根据 key 的值计算 hash 值，找到该元素在数组中存储的下标；
2. 如果数组是空的，则调用 resize 进行初始化；如果没有哈希冲突直接放在对应的数组下标里；
3. 如果冲突了，且 key 已经存在，就覆盖掉 value；
4. 如果冲突后，发现该节点是红黑树，就将这个节点挂在树上；
5. 如果冲突后是链表，判断该链表是否大于 8 ，如果大于 8 并且数组容量小于 64，就进行扩容；
6. 如果链表长度大于 8 并且数组的容量大于等于 64，则将这个结构转换为红黑树；否则，链表插入键值对，若 key 存在，就覆盖掉 value。

## HashTable
哈希表，存储的内容是键值对(key-value)映射。

Hashtable效率低下主要是因为其实现使用了synchronized关键字对put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁，也就是说在进行put等修改Hash表的操作时，锁住了整个Hash表，从而使得其表现的效率低下。

特性：
- 线程安全，所有公共的方法都使用了synchronized关键字。
- key、value值均不可为null。设计如此。

## TreeMap
Map接口的具体实现类，是一个有序的key-value集合，底层数据结构基于红黑树实现，插入操作比较复杂，不适合频繁修改的场景。

特性：
- 元素根据key按自然排序规则或自定义规则顺序排列。
- 线程不安全。
- 不允许null键，允许null值。TreeMap的put方法会根据key排列，调用compareTo方法，key为null时，会报空指针错。

## LinkedHashMap
LinkedHashMap实现了Map接口，继承自HashMap接口，底层数据结构使用双向链表和哈希表。这个双向链表决定了内部的键值对的迭代顺序，是键值对被插入哈希表的顺序。

特性：
- 元素按输入顺序排列。
- key、value允许null值。底层数据结构是双向链表，空键值不影响。
- 线程不安全。

## ConCurrentHashMap
ConCurrentHashMap是Map接口的具体实现类，为了解决HashMap线程不安全，而HashTable效率低下而生。
底层数据结构由双数组加链表实现，segment数组用于存储分段的哈希表，哈希表基于数组加链表实现。

JDK1.5 ~ 1.7中使用了分段锁机制实现ConcurrentHashMap.
ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。
Segment数组将整个Hash表划分为多个分段，每个Segment元素则类似于一个Hashtable，在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可实现多线程put操作。

Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

concurrencyLevel: 并行级别，默认是 16。就是`ConcurrentHashMap`有 16 个 `Segments`，所以理论上，这时最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。再具体到每个 Segment 内部，其实每个 Segment 很像 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

---

ConcurrentHashMap通过分段锁机制实现，所以其最大并发度受Segment的个数限制。
在JDK1.8中，ConcurrentHashMap的实现原理摒弃了这种设计，选择了与HashMap类似的数组+链表+红黑树的方式实现，而加锁则采用CAS和synchronized实现。



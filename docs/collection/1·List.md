# 1·List

- [1·List](#1list)
  - [List是什么](#list是什么)
    - [API](#api)
  - [集合是什么](#集合是什么)
  - [ArrayList](#arraylist)
    - [特性](#特性)
    - [数据结构](#数据结构)
    - [使用示例](#使用示例)
    - [源码](#源码)
      - [数据结构](#数据结构-1)
        - [构造函数](#构造函数)
      - [自动扩容](#自动扩容)
      - [add(), addAll()](#add-addall)
      - [set()](#set)
      - [get()](#get)
      - [remove()](#remove)
      - [trimToSize()](#trimtosize)
      - [indexOf(), lastIndexOf()](#indexof-lastindexof)
    - [Fail-Fast机制](#fail-fast机制)
  - [LinkedList](#linkedlist)
  - [Vector](#vector)

## List是什么
List接口继承自Collection接口，表示元素有序、可重复的链表容器集合。
List定义时申请计算机的内存是不连续的，每个元素存储了下一个下标元素的内存地址。因此添加元素的效率较高，可以使用下标访问元素，但查找某个下标的元素时效率较低，因为需要从头开始一直根据下一个元素的地址链式查询下去。

### API
| size() | 获取长度 |
| --- | --- |
| isEmpty() | 判断是否长度为0 |
| get() | 获取元素 |
| set() | 放入元素 |
| add() | 放入元素，时间开销跟插入位置有关 |
| addAll() | 放入全部元素，开销跟添加元素的个数成正比 |
| clear() | 清除所有元素 |
| contains() | 判断元素是否存在 |
| remove() | 删除一个元素 |
| toArray() | 转为数组 |

## 集合是什么
集合是一种容器，容器是容纳Java对象的对象。Java容器主要包括`Collection`和`Map`两个根接口。
由于在Java中数组容器的长度固定（定义后不可变），需要数组扩容，不方便使用且效率低。Java中提供了集合`Collection<E>`，其包括`List`，`Set`，`Queue`等接口。
容器里只能放对象，对于基本类型(int, long, float, double等)，需要将其包装成对象类型后(Integer, Long, Float, Double等)才能放到容器里。

## ArrayList
ArrayList对象实现了List接口，List接口继承自Collection接口。
是一个元素有序排列、可重复的链表容器。

### 特性
ArrayList特性：
1. 允许对元素进行快速随机访问。
2. 对头部和尾部进行添加元素时有很高的效率。
3. 从中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。
4. 它适合随机查找和遍历，不适合插入和删除。
5. 它按放入元素先后顺序排列（顺序容器），并允许放入`NULL`元素。

### 数据结构
ArrayList实现了List接口，底层数据结构通过数组`Object[]`实现，要求申请一段连续的内存地址。基于动态数组实现，可以看作是一个自动扩容的数组。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时进行扩容，将已经有数组的数据复制到新的存储空间中。
ArrayList实例的容量就是底层数组的实际大小，在实例化时可以指定其长度（不指定默认为10），当放入元素时容量不足时自动扩容底层数组。
为追求效率，`ArrayList`没有实现同步(synchronized)，如果多个线程并发访问，需要手动同步，也可用`Vector`替代。

### 使用示例
```java
public static void s1() {
    List list1 = new ArrayList<>();//默认存放Object类型
    List list2 = new ArrayList<String>();//<>中是集合存放的泛型，使集合中只能存放一类元素。
    //存放元素
    list2.add("集合");
    list2.add("list");
    list2.add("list");
    list2.add("list");
    list2.add("list");
    list2.add("string");
    list2.add(1,"lee");//在下标为1的地方插入元素
    //获取元素
    System.out.println(list2.get(1));
    //获取长度
    System.out.println(list2.size());
    //替换
    list2.set(3,"Object");
    //删除，若表中有重复元素，remove一次只能删一个匹配的元素
    list2.remove("list");

    //删除表中所有remove匹配的元素
    while(list2.indexOf("list")>=0){
        list2.remove("list");
    }
    //普通for循环遍历集合
    for (int i = 0; i < list2.size(); i++) {
        System.out.println(list2.get(i));
    }
    //foreach循环遍历集合
    for (Object s1 : list2) {
        System.out.println(s1);
    }
    System.out.println(list2);
    /*
    * 使用迭代器遍历集合
    * 先把集合中的元素导入到迭代器中
    * */
    Iterator<String> it = list2.iterator();
    //如果还有下一个就循环
    while(it.hasNext()){
        System.out.println(it.next());//输出下一个元素
    }
}
```
### 源码

#### 数据结构
```java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

##### 构造函数
```java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

#### 自动扩容
每次添加元素时，都会检查添加元素后的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个公开的方法ensureCapacity(int minCapacity)来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。
数组进行扩容时，会将老数组中的元素重新拷贝到新数组中，每次数组容量的增长大约是其原容量的1.5倍。这种操作的代价是很高的，因此在实际使用时，应该尽量避免数组容量的扩张。当可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。
```java
/**
 * Increases the capacity of this <tt>ArrayList</tt> instance, if
 * necessary, to ensure that it can hold at least the number of elements
 * specified by the minimum capacity argument.
 *
 * @param   minCapacity   the desired minimum capacity
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

#### add(), addAll()
这两个方法都是向容器中添加新元素，这可能会导致_capacity_不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过grow()方法完成的。
```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

/**
 * Inserts the specified element at the specified position in this
 * list. Shifts the element currently at that position (if any) and
 * any subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```
`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。
`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本。
一个是在末尾添加的`addAll(Collection<? extends E> c)`方法。
一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。
跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。
```java
/**
 * Appends all of the elements in the specified collection to the end of
 * this list, in the order that they are returned by the
 * specified collection's Iterator.  The behavior of this operation is
 * undefined if the specified collection is modified while the operation
 * is in progress.  (This implies that the behavior of this call is
 * undefined if the specified collection is this list, and this
 * list is nonempty.)
 *
 * @param c collection containing elements to be added to this list
 * @return <tt>true</tt> if this list changed as a result of the call
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

/**
 * Inserts all of the elements in the specified collection into this
 * list, starting at the specified position.  Shifts the element
 * currently at that position (if any) and any subsequent elements to
 * the right (increases their indices).  The new elements will appear
 * in the list in the order that they are returned by the
 * specified collection's iterator.
 *
 * @param index index at which to insert the first element from the
 *              specified collection
 * @param c collection containing elements to be added to this list
 * @return <tt>true</tt> if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

#### set()
既然底层是一个数组，_ArrayList_的set()方法也就变得非常简单，直接对数组的指定位置赋值即可。
```java
public E set(int index, E element) {
    rangeCheck(index);//下标越界检查
    E oldValue = elementData(index);
    elementData[index] = element;//赋值到指定位置，复制的仅仅是引用
    return oldValue;
}
```

#### get()
get()方法同样很简单，唯一要注意的是由于底层数组是Object[]，得到元素后需要进行类型转换。
```java
public E get(int index) {
    rangeCheck(index);
    return (E) elementData[index];//注意类型转换
}
```

#### remove()
remove()方法也有两个版本。
一个是remove(int index)删除指定位置的元素。
一个是remove(Object o)删除第一个满足o.equals(elementData[index])的元素。
删除操作是add()操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋null值。
```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
    return oldValue;
}
```
GC特别说明，**有了垃圾收集器并不意味着一定不会有内存泄漏**。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋null值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。

#### trimToSize()
ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能。
```java
/**
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an <tt>ArrayList</tt> instance.
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
            ? EMPTY_ELEMENTDATA
            : Arrays.copyOf(elementData, size);
    }
}
```

#### indexOf(), lastIndexOf()
获取元素的第一次出现的index
```java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
     */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
获取元素的最后一次出现的index
```java
/**
 * Returns the index of the last occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the highest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### Fail-Fast机制
ArrayList采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。


## LinkedList
LinkedList插入元素效率更快，基于双向链表实现，提供了一些对集合头部和尾部的操作，只能顺序访问，但是可以快速地在链表中间插入和删除元素，还可以用作栈、队列和双向队列。

- addFirst()：向队列头部添加元素。
- addLash()：向队列尾部添加元素。

```java
public void  testLinkedList(){
	LinkedList<Integer> ll = new LinkedList<Integer>();
	ll.addFirst(20);
	ll.addLast(66);
	ll.addFirst(10);
    ll.addLast(99);
    System.out.println(ll);
    System.out.println(ll.peek());//检索第一个元素
    System.out.println(ll.poll());//检索并删除第一个元素
    System.out.println(ll.pop());//删除并返回第一个元素
    System.out.println(ll);
}
```

## Vector
ArrayList 类似，但它是线程安全的。
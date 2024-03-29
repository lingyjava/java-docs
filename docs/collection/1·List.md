# 1·List

- [1·List](#1list)
  - [集合](#集合)
  - [List是什么](#list是什么)
  - [List特性](#list特性)
  - [链表内存模型](#链表内存模型)
  - [API](#api)
  - [ArrayList](#arraylist)
    - [自动扩容原理](#自动扩容原理)
    - [Fail-Fast机制](#fail-fast机制)
    - [更多资料](#更多资料)
  - [LinkedList](#linkedlist)
  - [Vector](#vector)

## 集合
集合是一种容器，容器是容纳Java对象的对象。容器里只能放对象，对于基本类型(int, long, float, double等)，需要将其包装成对象类型后(Integer, Long, Float, Double等)才能放到容器里。
Java容器主要包括`Collection`和`Map`两个根接口。
Java中数组的长度固定（定义后不可变），需要数组扩容，不方便使用且效率低。

## List是什么
List是链表对象接口，并继承了Collection接口。

## List特性
List的特性：
- List中的元素默认按输入顺序有序排列。
- List中的元素可重复。
- 初始容量不指定时的默认容量在JDK1.6后是10，在JDK1.8后是空的。
- 添加元素效率高，访问元素效率低。

## 链表内存模型
链表在创建时申请的计算机内存地址是不连续的，通过每个元素存储下一个元素的内存地址值实现链表结构。在添加元素时只需在上一个元素中记录新元素的内存地址值即可，查询元素时则需要以链式结构逐一向下查询。

## API
List接口定义的API，List的具体实现类（ArrayList、Vector、LinkedList）都必须实现以下方法：
| api        | 作用                                     |
| ---------- | :--------------------------------------- |
| size()     | 获取长度                                 |
| isEmpty()  | 判断是否长度为0                          |
| get()      | 获取元素                                 |
| set()      | 放入元素                                 |
| add()      | 放入元素，时间开销跟插入位置有关         |
| addAll()   | 放入全部元素，开销跟添加元素的个数成正比 |
| clear()    | 清除所有元素                             |
| contains() | 判断元素是否存在                         |
| remove()   | 删除一个元素                             |
| toArray()  | 转为数组                                 |

## ArrayList
ArrayList具体实现了List接口。

实现方式：
底层数据结构通过数组`Object[]`实现，要求申请一段连续的内存地址。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要进行扩容，将已经有数组的数据复制到新的存储空间中。当放入元素时容量不足时自动扩容底层数组，可以看作是一个自动扩容的数组。

为追求效率，`ArrayList`没有实现同步(synchronized)，如果多个线程并发访问，需要手动同步，也可用`Vector`替代。

特性：
- 查询快，增删慢（中间位置插入或者删除元素时，需要对数组进行复制、移动，代价比较高）。
- 按输入顺序有序排列。
- 允许放入`NULL`元素，ArrayList底层是数组，添加null并未对数据结构造成影响。

### 自动扩容原理
在每次添加元素时，检查添加元素后的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。

数组扩容通过一个公开的方法ensureCapacity(int minCapacity)来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

数组进行扩容时，会将老数组中的元素重新拷贝到新数组中，每次数组容量的增长大约是其原容量的1.5倍。这种操作的代价很高，因此在实际使用时，应该尽量避免数组容量的扩张。当可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。


### Fail-Fast机制
ArrayList采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

### 更多资料
- [Collection - ArrayList 源码解析](https://pdai.tech/md/java/collection/java-collection-ArrayList.html)

## LinkedList
LinkedList具体实现了List接口，是基于双向链表的实现。
提供了一些对集合头部和尾部的操作，可以快速地在链表中间插入和删除元素，还可以用作栈、队列和双向队列。

实现方式：
基于双向链表实现，申请的计算机内存是不连续的，每个元素中存储上一个元素和下一个元素的内存地址值，因此从任何一个地方插入元素效率都很高，查询元素时也可以从头部或尾部开始查询。

特性：
- 增删块，查询慢。
- 按输入顺序有序排列。
- 允许放入`NULL`元素，LinkedList底层为双向链表，node.value = null 没有问题。

| api        | 作用                 |
| ---------- | :------------------- |
| addFirst() | 向队列头部添加元素   |
| addLash()  | 向队列尾部添加元素   |
| peek()     | 检索第一个元素       |
| poll()     | 检索并删除第一个元素 |
| pop()      | 删除并返回第一个元素 |

## Vector
底层数据结构是数组Array，查询快。增删慢。
Vector实现了 synchronized ，这是Vector和ArrayList的唯一的区别。


# 3·Map

- [3·Map](#3map)
  - [Map是什么](#map是什么)
  - [HashMap](#hashmap)
    - [put方法的流程](#put方法的流程)
  - [HashTable](#hashtable)
  - [ConCurrentHashMap](#concurrenthashmap)

## Map是什么
Map集合可以存储两个对象（key-value）。

特性：
1. Map集合不能存放重复元素(key不能重复) 。
2.  如果key重复了，相当于覆盖。

## HashMap
HashMap基于哈希表实现Map接口，以key-value键值对形式存储数据，
JDK1.7 底层使用的是数组 + 链表。
JDK1.8 底层使用的是数组 + 链表 + 红黑树。

```java
public void testHashMap() {
    HashMap<String,Integer> hm = new HashMap<String,Integer>();
    //添加元素
    hm.put("张三",99);
    hm.put("李四",68);
    hm.put("王五",90);
    hm.put("赵六",55);
    //删除
    hm.remove("张三");
    Scanner sc = new Scanner(System.in);
    //取出所有key
    Set<String> keys = hm.keySet();
    for (String str:keys){
        System.out.print(str+"\t");
        System.out.println();
        //取出所有value
        //System.out.println(hm.get(str)+"\t");
    }
    System.out.println("请输入姓名查询成绩:");
    String name = sc.nextLine();
    System.out.println(hm.get(name));

    String str = "qwertyuqwertyuiopkjuio";
    HashMap<Character,Integer> hh = new HashMap<Character,Integer>();
    for (int i=0;i<str.length();i++){
        if (hh.get(str.charAt(i))!=null){
            hh.put(str.charAt(i),hh.get(str.charAt(i))+1);
        }else{
            hh.put(str.charAt(i),1);
        }
    }
    Set<Character> sc1 = hh.keySet();
    for (Character c:sc1){
        System.out.print(c+"出现"+hh.get(c)+"次"+"\t");
    }
    System.out.println();
}
```

### put方法的流程

1. 首先根据 key 的值计算 hash 值，找到该元素在数组中存储的下标；
2. 如果数组是空的，则调用 resize 进行初始化；如果没有哈希冲突直接放在对应的数组下标里；
3. 如果冲突了，且 key 已经存在，就覆盖掉 value；
4. 如果冲突后，发现该节点是红黑树，就将这个节点挂在树上；
5. 如果冲突后是链表，判断该链表是否大于 8 ，如果大于 8 并且数组容量小于 64，就进行扩容；
6. 如果链表长度大于 8 并且数组的容量大于等于 64，则将这个结构转换为红黑树；否则，链表插入键值对，若 key 存在，就覆盖掉 value。

## HashTable
类似于HashMap，但是线程安全的。

## ConCurrentHashMap
ConCurrentHashMap是Map接口的具体实现类，为了解决HashMap线程不安全，而HashTable效率低下而生。
其实现方式在JDK1.7和1.8版本略有不同。
JDK1.7底层使用数组（Segment）和链表（HashEntry）

# Set

- [Set](#set)
  - [Set是什么](#set是什么)
  - [Set特性](#set特性)
  - [HashSet](#hashset)
  - [LinkedHashSet](#linkedhashset)
  - [TreeSet](#treeset)
    - [自定义排序规则](#自定义排序规则)
  - [ArraySet](#arrayset)
  - [利用Set去重](#利用set去重)

## Set是什么
Set是集合接口，继承自Collection接口。

## Set特性
- 元素无序排列。
- 元素不可重复。

## HashSet
HashSet是Set接口的具体实现类，数据结构基于哈希表。
HashSet本质还是在内部维护一个HashMap对象，将所有的数据都交给HashMap进行处理。

特性：
- 元素无序排列。
- 元素不可重复。依赖两个方法来保证元素唯一性：`hashCode()`和`equals()`.
- 可以存null值。底层是HashMap，可以有1个为null的元素。
- 没有实现同步，线程不安全。

## LinkedHashSet
LinkedHashSet是Set接口的具体实现类，数据结构基于双向列表和哈希表。
LinkedHashSet是HashSet的子类，是对LinkedHashMap包装了一层。
主要适用于对于元素的添加顺序读取有要求的场景，比如FIFO这样的场景。插入性能略低于HashSet，但在迭代访问set里面的全部元素时有很好的性能。

特性：
- 元素有序排列，底层维护了一个数组+双向链表，根据元素的hashCode值来决定元素的存储位置，同时使用链表维护元素的次序, 使元素看起来是以插入顺序保存的。
- 元素不可重复，基于哈希表实现元素唯一。
- 可以存储null值。LinkHashSet底层是LinkedHashMap，允许存在一个为null的元素
- 线程不安全

## TreeSet
TreeSet是Set接口的具体实现类，数据结构基于红黑树。TreeSet是基于TreeMap实现。
add、remove、search等操作时间复杂度为O(log n)，效率不如`HashSet`的O(1)。

特性：
- 元素有序排列，根据树结构实现有序性，不按输入顺序排列，默认是按自然顺序排列，也可以自定义排序规则，根据比较器确认排序顺序。
- 元素不可重复，根据比较的返回值是否是0来保证元素唯一性。
- JDK8后元素不允许设置null值。TreeSet不能有key为null的元素，会报NullPointerException.
- 线程不安全。

### 自定义排序规则
`TreeSet`集合添加对象时，如果元素无法按自然顺序排列时报错，自定义排序规则示例如下。
- 在实体类中实现`Comparable`接口，并实现`compareTo`方法。
```java
// 实体类中
public class Person implements Comparable{
	/*
     *  返回值规则：
     *  负数：表示后面元素大
     *  正数：表示前面元素大
     *  0：相等
     * */
    @Override
    public int compareTo(Object o) {
        if (this ==o ){
            // 内存地址值相等,同一对象,相等
            return 0;
        }
        /*
        * 比较分数，年龄相等比较姓名。
        * */
        if (o instanceof Person){
            if (this==o){
                return 0;
            }
            Person p = (Person)o;
            if (this.grade < p.getGrade()){
                return -1;
            }else if (this.grade > p.getGrade()){
                return 1;
            }else{
                //分数相等,比较年龄
                if (this.getAge()<p.getAge()){
                    return -1;
                }else if (this.getAge()>p.getAge()){
                    return 1;
                }else{
                    //年龄相等,比较姓名
                    return this.getName().compareTo(p.getName());
                }
            }
        }
        return 0;
    }
}
```

## ArraySet
Android.util包下的类，底层数据结构是双数组，有序。实时扩容，节约内存。可以代替使用HashSet。
ArraySet通过调用System中提供的一个native静态方法arraycopy()，实现数组之间的复制。采取一种用时间换取内存空间的优化思路，其元素扩容是时刻变化的，也就是说会随时根据内容动态调整数组的大小。
因为每次操作都要实现数组复制，会影响到元素操作的效率。理论上来说，在大数据量的情况下，更频繁的数据条数大幅度变化下，效率会变低。但实际上，发现其速度在数万条数据的情况下，相差无几。

## 利用Set去重  
```java
// 统计字符出现的次数
public void statistics1(){
    String s1 = "leeLuoUpperWater";
    //创建HashSet集合
    HashSet<Character> hs = new HashSet<Character>();
    //将字符串插入到Set集合，利用Set集合去重复的特性找到出现过的字符
    for (int i = 0; i < s1.length(); i++) {
        hs.add(s1.charAt(i));
    }
    //用出现过的字符与字符串对比
    for (Character s2 : hs) {
        int num = 0;
        for (int i = 0; i < s1.length(); i++) {
            if (s2.equals(s1.charAt(i))){
                num++;
            }
        }
        System.out.println(s2+"出现"+num+"次");
    }
}
```

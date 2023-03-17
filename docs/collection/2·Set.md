# Set

- [Set](#set)
  - [TreeSet](#treeset)
    - [自定义排序规则](#自定义排序规则)
    - [使用示例](#使用示例)
  - [HashSet](#hashset)
  - [LinkedHashSet](#linkedhashset)
  - [利用Set去重](#利用set去重)
    - [统计字符串中每个字符出现的次数。](#统计字符串中每个字符出现的次数)

## TreeSet
TreeSet是Set接口的具体实现类，数据结构基于红黑树。具有Set接口元素不重复的特性。并且根据树结构，实现有序性，默认按自然顺序排列（123...，abc...），也可以自定义排序规则。
可以根据范围查找元素，时间复杂度为O(logN) ，效率不如`HashSet`的O(1)。

### 自定义排序规则
往`TreeSet`集合添加对象，由于无法按自然顺序排列，所以会报错，对象需要自定义排序规则。
在实体类中实现`Comparable`接口，并实现`compareTo`方法。
```java
// 实体类案例
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

### 使用示例
```java
public void testTreeSet(){
    TreeSet<Integer> ts1 = new TreeSet<Integer>();
    ts1.add(3);
    ts1.add(4);
    ts1.add(1);
    ts1.add(2);
    System.out.println(ts1);

    TreeSet<Person> ts2 = new TreeSet<Person>();
    Person p1 = new Person();
    p1.setName("法外狂徒");
    p1.setAge(22);
    p1.setGrade(66);

    Person p2 = new Person();
    p2.setName("李四");
    p2.setAge(18);
    p2.setGrade(77);

    ts2.add(p1);
    ts2.add(p2);
    System.out.println(ts2);
}
```

## HashSet
HashSet：里面元素是根据元素的内存地址值进行排列的。
基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

特性：
1. 无序(不会按照存放顺序排列，没有下标)。
2. 不能存放重复元素。

## LinkedHashSet
具有 HashSet 的查找效率，内部使用双向链表维护元素的插入顺序。

## 利用Set去重

### 统计字符串中每个字符出现的次数。  
```java
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
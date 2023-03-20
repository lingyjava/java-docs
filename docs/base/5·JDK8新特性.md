# 5·JDK8新特性

- [5·JDK8新特性](#5jdk8新特性)
  - [函数式编程](#函数式编程)
  - [求值方式](#求值方式)
  - [lambda表达式](#lambda表达式)
    - [目标类型](#目标类型)
    - [变量捕获 \& 限制](#变量捕获--限制)
    - [方法引用](#方法引用)
    - [作用原理](#作用原理)
  - [stream \& parallelStream](#stream--parallelstream)
    - [parallelStream原理](#parallelstream原理)
    - [性能测试](#性能测试)
  - [匿名内部类](#匿名内部类)
  - [forEach](#foreach)
  - [方法引用](#方法引用-1)
  - [Filter \& Predicate](#filter--predicate)
  - [Map\&Reduce](#mapreduce)
  - [Collectors](#collectors)
  - [flatMap](#flatmap)
  - [distinct](#distinct)
  - [count](#count)
  - [Match](#match)
  - [min,max,summaryStatistics](#minmaxsummarystatistics)
  - [peek](#peek)
  - [数值流](#数值流)
  - [创建流](#创建流)
  - [使用示例](#使用示例)
    - [筛选排序](#筛选排序)

## 函数式编程
> 面向对象编程是对数据进行抽象，而函数式编程是对行为进行抽象。

函数式编程的好处，编写更易阅读的代码，代码更多地表达了业务逻辑，而不是从机制上如何实现。易读的代码也易于维护、更可靠、更不容易出错。
核心思想：使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

## 求值方式
**惰性求值：**这行代码并未做什么实际性的工作，filter只是描述了Stream，没有产生新的集合。
`lists.stream().filter(f -> f.getName().equals("p1"));`
**及早求值：**collect最终会从Stream产生新值，拥有终止操作。
`List<Person> list2 = lists.stream().filter(f -> f.getName().equals("p1")).collect(Collectors.toList());`

> 理想方式是形成一个惰性求值的链，最后用一个及早求值的操作返回想要的结果。与建造者模式相似，建造者模式先是使用一系列操作设置属性和配置，最后调用build方法，创建对象。

## lambda表达式
Lambda表达式在Java中又称为闭包或匿名函数。

### 目标类型
lambda表达式仅能放入如下代码，这些称为lambda表达式的目标类型，可以用作返回类型，或lambda目标代码的参数：

- 预定义使用了 `@Functional` 注释的函数式接口。
- 自带一个抽象函数的方法。
- SAM(Single Abstract Method 单个抽象方法)类型。

若一个方法接收Runnable、Comparable或者 Callable 接口，都有单个抽象方法，可以传入lambda表达式。类似的，如果一个方法接受声明于 java.util.function 包内的接口，例如 Predicate、Function、Consumer 或 Supplier，那么可以向其传lambda表达式。

### 变量捕获 & 限制
lambda内部可以使用静态、非静态和局部变量。

lambda表达式限制只能引用 final 或 final 局部变量，不能在lambda内部修改定义在域外的变量。只是访问它而不作修改是可以的。
```java
List<Integer> primes = Arrays.asList(new Integer[]{2, 3,5,7});
int factor = 2;
primes.forEach(element -> { factor++; });
// 结果：Compile time error : "local variables referenced from a lambda expression must be final or effectively final"

```

### 方法引用
lambda表达式内可以使用方法引用，仅当该方法不修改lambda表达式提供的参数。若对参数有任何修改，则不能使用方法引用，而需键入完整地lambda表达式。
```java
//  修改了lambda提供的参数(可以省略类型声明，编译器可以从列表的类属性推测出来)
list.forEach((String s) -> System.out.println("*" + s + "*"));

list.forEach(n -> System.out.println(n)); 
// 未修改lambda提供的参数，使用方法引用
list.forEach(System.out::println);  

```
### 作用原理
Lambda方法在编译器内部被翻译成私有方法，并派发 invokedynamic 字节码指令来进行调用。可以使用JDK中的 javap 工具来反编译class文件。使用 javap -p 或 javap -c -v 命令来看一看lambda表达式生成的字节码：`private static java.lang.Object lambda$0(java.lang.String); `

## stream & parallelStream
> 每个Stream都有两种模式: 顺序执行和并行执行。

**顺序流：**使用顺序方式遍历时，每个item读完后再读下一个item。
`List <Person> people = list.getStream.collect(Collectors.toList()); `
**并行流：**使用并行去遍历时，数组会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。
`List <Person> people = list.getStream.parallel().collect(Collectors.toList());`

### parallelStream原理
核心思想就是大而化小，分配到不同机器去运行，最终将所有机器的结果结合起来得到一个最终结果，Stream则是利用多核技术可将大数据通过多核并行处理。
```java
List originalList = someData;
// 将数据分小部分
split1 = originalList(0, mid);
split2 = originalList(mid,end);
// 小部分执行操作
new Runnable(split1.process());
new Runnable(split2.process());
// 将结果合并
List revisedList = split1 + split2;
```

### 性能测试
如果是多核机器，理论上并行流则会比顺序流快上一倍。
```java
public static void main(String[] args) {
    long t0 = System.nanoTime();
    //初始化一个范围100万整数流,求能被2整除的数字，toArray()是终点方法
    int a[]= IntStream.range(0, 1_000_000).filter(p -> p % 2==0).toArray();
    long t1 = System.nanoTime();

    int b[]=IntStream.range(0, 1_000_000).parallel().filter(p -> p % 2==0).toArray();
    long t2 = System.nanoTime();

    System.out.printf("serial: %.2fs, parallel %.2fs%n", (t1 - t0) * 1e-9, (t2 - t1) * 1e-9);
    // serial: 0.05s, parallel 0.02s
}
```

## 匿名内部类
```java
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();

// 用法
(params) -> expression
(params) -> statement
(params) -> { statements }
```

## forEach
```java
// forEach
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(n -> System.out.println(n));
 
// 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
features.forEach(System.out::println);
```

## 方法引用

- 构造引用
```java
// Supplier<Student> s = () -> new Student();
Supplier<Student> s = Student::new;
```

- 对象::实例方法，Lambda表达式的(形参列表)与实例方法的(实参列表)类型，个数是对应。
```java
// set.forEach(t -> System.out.println(t));
set.forEach(System.out::println);
```

- _类名::静态方法_
```java
// Stream<Double> stream = Stream.generate(() -> Math.random());
Stream<Double> stream = Stream.generate(Math::random);
```

- _类名::实例方法_
```java
//  TreeSet<String> set = new TreeSet<>((s1,s2) -> s1.compareTo(s2));
/*  这里如果使用第一句话，编译器会有提示: Can be replaced with Comparator.naturalOrder，这句话告诉我们
  String已经重写了compareTo()方法，在这里写是多此一举，这里为什么这么写，是因为为了体现下面
  这句编译器的提示: Lambda can be replaced with method reference。好了，下面的这句就是改写成方法引用之后: 
*/
TreeSet<String> set = new TreeSet<>(String::compareTo);
```

## Filter & Predicate
常规用法
```java
public static void main(args[]){
    List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");

    System.out.println("Languages which starts with J :");
    filter(languages, (str)->str.startsWith("J"));

    System.out.println("Languages which ends with a ");
    filter(languages, (str)->str.endsWith("a"));

    System.out.println("Print all languages :");
    filter(languages, (str)->true);

    System.out.println("Print no language : ");
    filter(languages, (str)->false);

    System.out.println("Print language whose length greater than 4:");
    filter(languages, (str)->str.length() > 4);
}

public static void filter(List names, Predicate condition) {
    names.stream().filter((name) -> (condition.test(name))).forEach((name) -> {
        System.out.println(name + " ");
    });
}
```

- 多个Predicate组合filter
```java
// 可以用and()、or()和xor()逻辑函数来合并Predicate，
// 例如要找到所有以J开始，长度为四个字母的名字，你可以合并两个Predicate并传入
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
    .filter(startsWithJ.and(fourLetterLong))
    .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));
```

## Map&Reduce

- map用来**归类**，结果一般是一组数据，比如可以将list中的学生分数映射到一个新的stream中。
- reduce用来**计算值**，结果是一个值，比如计算最高分。
```java
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream().map((cost) -> cost + .12*cost).reduce((sum, cost) -> sum + cost).get();
System.out.println("Total : " + bill);

public class StreamTest02 {
 
    public static void main(String[] args) {
        List<Student> list = InitData.getStudent();
        //省略初始化List数据
        //使用map方法获取list数据中的name
        List<String> names = list.stream().map(Student::getName).collect(Collectors.toList());
        System.out.println(names);
 
        //使用map方法获取list数据中的name的长度
        List<Integer> length = list.stream().map(Student::getName).map(String::length).collect(Collectors.toList());
        System.out.println(length);
 
        //将每人的分数-10
        List<Integer> score = list.stream().map(Student::getScore).map(i -> i - 10).collect(Collectors.toList());
        System.out.println(score);
 
        //计算学生总分
        Integer totalScore1 = list.stream().map(Student::getScore).reduce(0,(a,b) -> a + b);
        System.out.println(totalScore1);
 
        //计算学生总分，返回Optional类型的数据，改类型是java8中新增的，主要用来避免空指针异常
        Optional<Integer> totalScore2 = list.stream().map(Student::getScore).reduce((a,b) -> a + b);
        System.out.println(totalScore2.get());
 
        //计算最高分和最低分
        Optional<Integer> max = list.stream().map(Student::getScore).reduce(Integer::max);
        Optional<Integer> min = list.stream().map(Student::getScore).reduce(Integer::min);
 
        System.out.println(max.get());
        System.out.println(min.get());
    }
}
```

## Collectors
- Collectors.joining(", ")
- Collectors.toList()
- Collectors.toSet() ，生成set集合
- Collectors.toMap(MemberModel::getUid, Function.identity())
- Collectors.toMap(ImageModel::getAid, o -> IMAGE_ADDRESS_PREFIX + o.getUrl())
```java
// 将字符串换成大写并用逗号链接起来
List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
String G7Countries = G7.stream().map(x -> x.toUpperCase()).collect(Collectors.joining(", "));
System.out.println(G7Countries);
```

## flatMap
将多个Stream连接成一个Stream.
```java
List<Integer> result= Stream.of(Arrays.asList(1,3),Arrays.asList(5,6)).flatMap(a->a.stream()).collect(Collectors.toList());
// 结果: [1, 3, 5, 6]
```

## distinct
去重。
```java
List<LikeDO> likeDOs=new ArrayList<LikeDO>();
List<Long> likeTidList = likeDOs.stream().map(LikeDO::getTid)
                .distinct().collect(Collectors.toList());
```

## count
计总数。
```java
int countOfAdult=persons.stream()
                       .filter(p -> p.getAge() > 18)
                       .map(person -> new Adult(person))
                       .count();
```

## Match
```java
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
```

## min,max,summaryStatistics
最小值，最大值。
```java
List<Person> lists = new ArrayList<Person>();
lists.add(new Person(1L, "p1"));
lists.add(new Person(2L, "p2"));
lists.add(new Person(3L, "p3"));
lists.add(new Person(4L, "p4"));
Person a = lists.stream().max(Comparator.comparing(t -> t.getId())).get();
System.out.println(a.getId());
```

- 如果比较器涉及多个条件，比较复杂，可以定制
```java
 Person a = lists.stream().min(new Comparator<Person>() {

      @Override
      public int compare(Person o1, Person o2) {
           if (o1.getId() > o2.getId()) return -1;
           if (o1.getId() < o2.getId()) return 1;
           return 0;
       }
 }).get();
```

- summaryStatistics.
```java
//获取数字的个数、最小值、最大值、总和以及平均值
List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
System.out.println("Highest prime number in List : " + stats.getMax());
System.out.println("Lowest prime number in List : " + stats.getMin());
System.out.println("Sum of all prime numbers : " + stats.getSum());
System.out.println("Average of all prime numbers : " + stats.getAverage());
```

## peek
peek主要被用在debug用途。
```java
List<Person> lists = new ArrayList<Person>();
lists.add(new Person(1L, "p1"));
lists.add(new Person(2L, "p2"));
lists.add(new Person(3L, "p3"));
lists.add(new Person(4L, "p4"));
System.out.println(lists);

List<Person> list2 = lists.stream()
				 .filter(f -> f.getName().startsWith("p"))
                .peek(t -> {
                    System.out.println(t.getName());
                })
                .collect(Collectors.toList());
System.out.println(list2);
```

## 数值流
如果使用流来进行求和操作，实现如下：
```java
Optional<Integer> totalScore2 = list.stream().map(Student::getScore)
                                    .reduce((a,b) -> a + b);
```
在java8中新增了三个原始类型流（IntStream、DoubleStream、LongStream）。
```java
//数值流
public class StreamTest03 {
    public static void main(String[] args) {
        List<Student> list = InitData.getStudent();
 
        //将stream转换为IntStream
        int totalScore = list.stream().mapToInt(Student::getScore).sum();
        System.out.println(totalScore);
 
        //计算平均分
        OptionalDouble avgScore = list.stream().mapToInt(Student::getScore).average();
        System.out.println(avgScore.getAsDouble());
 
        //生成1~100之间的数字
        IntStream num = IntStream.rangeClosed(1, 100);
 
        //计算1~100之间的数字中偶数的个数
        long count = IntStream.rangeClosed(1, 100).filter(n -> n%2 == 0).count();
        System.out.println(count);
    }
}
```

## 创建流
除了上面的流之外，还可以自己创建流。下面代码中展示了三种创建流的方式：
```java

//使用Stream.of创建流
Stream<String> str =  Stream.of("i","love","this","game");
str.map(String::toUpperCase).forEach(System.out::println);

//使用数组创建流
int[] num = {2,5,9,8,6};
IntStream intStream = Arrays.stream(num);
int sum = intStream.sum();//求和
System.out.println(sum);

//由函数生成流，创建无限流
Stream.iterate(0, n -> n+2).limit(10).forEach(System.out::println);
```

## 使用示例

### 筛选排序
列出90分以上的学生姓名，并按照分数降序排序。
```java

public class StreamTest01 {
    public static void main(String[] args) {
        List<Student> stuList = new ArrayList<>(10);
        stuList.add(new Student("刘一", 85));
        stuList.add(new Student("陈二", 90));
        stuList.add(new Student("张三", 98));
        stuList.add(new Student("李四", 88));
        stuList.add(new Student("王五", 83));
        stuList.add(new Student("赵六", 95));
        stuList.add(new Student("孙七", 87));
        stuList.add(new Student("周八", 84));
        stuList.add(new Student("吴九", 100));
        stuList.add(new Student("郑十", 95));
 
        //以前的写法，代码较多，每个操作都需要遍历集合
        List<Student> result1 = new ArrayList<>(10);
        //遍历集合获取分数大于90以上的学生并存放到新的List中
        for(Student s : stuList){
            if(s.getScore() >= 90){
                result1.add(s);
            }
        }
        //对List进行降序排序
        result1.sort(new Comparator<Student>(){
            @Override
            public int compare(Student s1, Student s2) {
                //降序排序
                return Integer.compare(s2.getScore(), s1.getScore());
            }
        });
        System.out.println(result1);
 
        //使用stream的写法
        /*
         * 1.获取集合的stream对象
         * 2.使用filter方法完成过滤
         * 3.使用sort方法完成排序
         * 4.使用collect方法将处理好的stream对象转换为集合对象
         */
        result1 = stuList.stream()
                .filter(s -> s.getScore()>=90)
                //.sorted((s1,s2) -> Integer.compare(s2.getScore(), s1.getScore()))
                //使用Comparator中的comparing方法
                .sorted(Comparator.comparing(Student :: getScore).reversed())
                .collect(Collectors.toList());
        System.out.println(result1);
    }
}
```

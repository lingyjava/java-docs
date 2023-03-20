# 7·Java泛型

- [7·Java泛型](#7java泛型)
  - [什么是泛型](#什么是泛型)
  - [泛型通配符](#泛型通配符)
  - [泛型类](#泛型类)
  - [泛型接口](#泛型接口)
  - [泛型方法](#泛型方法)
  - [无界通配符](#无界通配符)
  - [上界通配符](#上界通配符)
  - [下界通配符](#下界通配符)

## 什么是泛型
Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“**伪泛型**”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“**类型擦除**”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。所以泛型只在编译阶段有效。

---

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

## 泛型通配符
通配符是编码时约定的东西。泛型`T`可以换成`A ~ Z`任何一个字母都可以，换成任何字母都能代替，在可读性上会弱一些。

| T | （type）一个Java类型 |
| --- | --- |
| K | （key）代表Java键值对中的Key |
| V | （value）代表Java键值对中的Value |
| E | （element）代表Element |
| ? | 表示不确定的 Java 类型 |

## 泛型类
泛型类是指给类一个泛型，在实例化该对象时，必须指定其具体类型。
```java
public class Generic<T>{ 
    // 类型为T, T的类型由实例化时指定  
    private T key;

    public Generic(T key) { this.key = key;}

    public T getKey() {return key;}
}
```

## 泛型接口
泛型接口是指给接口一个泛型。
```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
// 实现泛型接口,未传入泛型实参

// 编译器会报错："Unknown class"
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {return null;}
}
```

---

实现泛型接口时，如果不传入泛型实参，需将泛型的声明也一起加到类中。
如果不声明泛型，编译器报错`"Unknown class"`
```java
// class FruitGenerator implements Generator<T>   "Unknown class"
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {return null;}
}
```

---

实现泛型接口时，传入泛型实参时，可以为T传入无数个实参，形成无数种类型的接口。
如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型。

```java
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    // public T next(); T替换成传入的String类型
    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

## 泛型方法
泛型方法，是在调用方法的时候指明泛型的具体类型 。
```java
/**
 * public 与 返回值中间 <T> 是指返回值为泛型 , 只有声明了<T>的方法才是泛型方法。
 * 泛型类中的使用了泛型的成员方法不属于泛型方法。
 */
public <T> T genericMethod(Class<T> tClass) throws InstantiationException ,
  IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
}
// 调用
Object obj = genericMethod(Class.forName("com.test.test"));
```

## 无界通配符
`?`作为无界通配符，当具体类型不确定的时候，使用通配符`?`，当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用`?`通配符来表未知类型。
`?`代替具体的实参类型，而不是形参类型，`?`和`Number、String、Integer`一样都是一种实际类型，可以把`?`看成所有类型的父类。是一种真实的类型。

```java
public void showKeyValue(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

Generic<Number> gNumber = new Generic<Number>(123);
showKeyValue(gNumber);

Generic<Integer> gInteger = new Generic<Integer>(456);
showKeyValue(gInteger); // 这个方法编译器会报错：Generic<java.lang.Integer> 
						// cannot be applied to Generic<java.lang.Number>
// 同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

// 将showKeyValue方法入参泛型改成?通配符,则两个方法都不会报错。
public void showKeyValue(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

## 上界通配符 
`< ? extends E >`上界通配符，上界用`extends`关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的子类。

- 如果传入的类型不是 E 或者 E 的子类，编译不成功。
- 泛型中可以使用 E 的方法。

```java
private <K extends A, E extends B> E test(K arg1, E arg2){
    E result = arg2;
    arg2.compareTo(arg1);
    //.....
    return result;
}
```

## 下界通配符
`< ? super E >`下界通配符，下界用`super`进行声明，表示参数化的类型可能是所指定的类型，或者是此类型的父类，直至`Object`。

- 在类型参数中使用 super 表示这个泛型中的参数必须是 E 或者 E 的父类。

```java
private static <T> void test(List<? super T> dst, List<T> src){
    for (T t : src) {
        dst.add(t);
    }
}
 
public static void main(String[] args) {
    List<Dog> dogs = new ArrayList<>();
    List<Animal> animals = new ArrayList<>();
    test(animals, dogs);
}

// Dog 是 Animal 的子类
class Dog extends Animal {}
class Animal{}
```

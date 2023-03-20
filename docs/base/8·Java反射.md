# 8·Java反射

- [8·Java反射](#8java反射)
  - [什么是反射](#什么是反射)
  - [类加载过程](#类加载过程)
  - [API](#api)
    - [获取class对象](#获取class对象)
    - [获取对象属性](#获取对象属性)
    - [获取对象方法](#获取对象方法)
  - [使用示例](#使用示例)

## 什么是反射
反射（Reflection）是通过方法获得class对象取得类的构造。通过反射可以获取任意一个类的所有属性和方法，还可以调用这些方法和属性，通过统一的语法创建对象获取对象。
反射是在运行时才知道要操作的类是什么，并在运行时获取类的完整构造，调用对应的方法。使用反射提高程序的通用性。所以反射也被称为框架的灵魂，可以利用反射机制获得类似动态语言的特性，使Java有一定的动态性，赋予了我们在运行时分析类以及执行类中方法的能力。

## 类加载过程
在JVM加载完Class文件后，会给每个类生成一个唯一的Class对象。该Class对象是该类所有实例共享的一个对象，包含与类有关的信息，在Class对象中会把属性以及方法都单独封装成一个对象。

三步骤：
1.  加载：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代表这个类的java.lang.Class对象。 
2.  链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。
验证：确保加载的类信息符合JVM规范，没有安全问题。
准备：正式为类变量(static)分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配。
解析：虚拟机常量池内的符合引用(常量名)替换为直接引用(地址)的过程。 
3.  初始化:到了此阶段，才真正开始执行类中定义的java程序代码，用于执行该类的静态初始器和静态初始块，如果该类有父类，优先对其父类进行初始化。 

## API

### 获取class对象
一个类在内存中只有一个Class对象。一个类被加载后，类的整个结构都会被封装在class对象中。  

---

1. 通过实例获取。
```java
Animal animal = new Animal();
Class animalClass = animal.getClass();
// 通过反射创建一个类的实例
Object obj = animalClass.newInstance;
```

2. 通过对象获取。
```java
Class animalClass = Animal.class;
Object obj = animalClass.newInstance;
```

3. 通过forName限定性类型获取。
```java
String className = "包名.类名"
Class animalClass3 = Class.forName(className);
Object obj = animalClass.newInstance;
```

### 获取对象属性  
```java
// 获取所有的公共属性
Field[] fs1 = animalClass.getFields();
// 获取所有属性
Field[] fs2 = animalClass.gerDeclaredFields();

// 根据属性名称获取属性  
Field[] ageField = animalClass.gerDeclaredFields("age");
// 取消安全检查，允许获取私有的属性值
ageField.setAccessible(true);
ageField.setInt(obj,999);
// 获取
int age = ageField.getInd(obj);
System.out.println(age);
```

### 获取对象方法
```java
// 获取所有的公共方法
Method[] ms = animalClass.getMethods();
// 获取所有方法
Method[] ms = animalClass.getDeclaredMethods();
Method eatMethod = animalClass.getMethod("eat",int.class,String.class);
eatMethod.invoke(obj,20,"lee");
Class<?>[] pcType = eatMethod.getParameterTypes();
```

## 使用示例
```java
public void r1(){
    String personStr = "com.example.bean.Person";//对象类型属性
    try {
        Class personClass = Class.forName(personStr);
        Object obj = personClass.newInstance();
        String parKey = "name";
        String parValue = "张三";
        
		//通过字符串拼接创建set方法
        String setParKey = "set"+parKey.substring(0,1).toUpperCase()+parKey.substring(1);
        Method setNameMethod = personClass.getMethod(setParKey,parValue.getClass());

        setNameMethod.invoke(obj,parValue);

        //通过字符串拼接创建get方法
        String getParKey = "get"+parKey.substring(0,1).toUpperCase()+parKey.substring(1);
        Method getNameMethod = personClass.getMethod(setParKey,parValue.getClass());

        Object result = getNameMethod.invoke(obj);

        System.out.println(result);

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
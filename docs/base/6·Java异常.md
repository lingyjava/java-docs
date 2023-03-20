# Java异常

- [Java异常](#java异常)
  - [什么是异常](#什么是异常)
  - [Throwable](#throwable)
  - [Error](#error)
  - [Exception](#exception)
    - [运行时异常](#运行时异常)
    - [非运行时异常（编译时异常）](#非运行时异常编译时异常)
    - [不可察异常](#不可察异常)
  - [API](#api)
  - [关键字](#关键字)
  - [抛出异常](#抛出异常)
    - [声明](#声明)
    - [抛出](#抛出)
  - [捕获异常](#捕获异常)
    - [try-catch-finally](#try-catch-finally)
    - [try-finally](#try-finally)
    - [try-with-resource](#try-with-resource)
  - [自定义异常](#自定义异常)
  - [常用异常](#常用异常)
    - [RuntimeException](#runtimeexception)
    - [IOException](#ioexception)
    - [其他](#其他)
  - [异常实践经验](#异常实践经验)
    - [非必要不使用](#非必要不使用)
    - [释放资源方式](#释放资源方式)
    - [尽量使用标准异常](#尽量使用标准异常)
    - [文档说明异常](#文档说明异常)
    - [具体异常优先捕获](#具体异常优先捕获)
    - [不捕获Throwable](#不捕获throwable)
    - [不忽略异常](#不忽略异常)
    - [不又记录又抛出](#不又记录又抛出)
    - [处理异常才捕获](#处理异常才捕获)
    - [不抛弃原始异常](#不抛弃原始异常)
    - [异常不控制流程](#异常不控制流程)
    - [finally块中不使用return。](#finally块中不使用return)
  - [JVM异常处理机制](#jvm异常处理机制)
    - [try catch -finally](#try-catch--finally)
    - [Return 和finally的问题](#return-和finally的问题)
  - [异常为什么耗时](#异常为什么耗时)
    - [异常发生瞬间](#异常发生瞬间)

## 什么是异常
Java异常都是对象，是`Throwable`子类的实例，描述出现在一段编码中的错误条件。当条件生成时，错误引发异常。异常的在程序中可以是抛出或者捕捉并处理，增强程序的健壮性。

## Throwable
`Throwable`是所有异常的超类，包含两个子类：`Error`（错误）和 `Exception`（异常）。它们用于指示发生了异常情况。
包含其线程创建时执行堆栈的快照，并提供了`printStackTrace()`等接口用于获取堆栈跟踪数据等信息。

## Error
`Error`指程序运行中出现的严重错误，包含两个子类：`IOException`和`RuntimeException`。用来表示运行应用程序中出现了严重的错误，是程序中无法处理的错误。

**不受检异常：**（此类错误一般表示代码运行时 JVM 出现问题，JVM 将终止线程，程序不会从错误中恢复。）

| Virtual MachineError | 虚拟机运行错误 |
| --- | --- |
| NoClassDefFoundError | 类定义错误 |
| OutOfMemoryError | 内存不足错误 |
| StackOverflowError | 栈溢出错误 |

## Exception
`Exception`是所有异常的父类，它分为两类：运行时异常、编译时异常。
属于程序本身可以捕获并且可以处理的异常。

### 运行时异常
`RuntimeException`及其子类指运行时异常。这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。
运行时异常的特点是Java编译器不会检查它，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

| NullPointerException | 空指针异常 |
| --- | --- |
| IndexOutOfBoundsException | 下标越界异常 |

### 非运行时异常（编译时异常）
又称为可查异常，是除了`RuntimeException`及其子类的异常。包括`IOException`、`SQLException`、`ParseException`以及它们的子类。
这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。也就是编译器要求必须处置的异常。

| ClassNotFoundException | 应用程序试图加载类时，找不到相应的类，抛出该异常。 |
| --- | --- |
| CloneNotSupportedException | 当调用 Object 类中的 clone 方法克隆对象，但该对象的类无法实现 Cloneable 接口时，抛出该异常。 |
| IllegalAccessException | 拒绝访问一个类的时候，抛出该异常。 |
| InstantiationException | 当试图使用 Class 类中的 newInstance 方法创建一个类的实例，而指定的类对象因为是一个接口或是一个抽象类而无法实例化时，抛出该异常。 |
| InterruptedException | 一个线程被另一个线程中断，抛出该异常。 |
| NoSuchFieldException | 请求的变量不存在 |
| NoSuchMethodException | 请求的方法不存在 |

### 不可察异常
不可察异常包括运行时异常和错误。编译器不要求强制处置的异常。

| ArithmeticException | 当出现异常的运算条件时，抛出此异常。例如，一个整数"除以零"时，抛出此类的一个实例。 |
| --- | --- |
| ArrayIndexOutOfBoundsException | 用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引。 |
| ArrayStoreException | 试图将错误类型的对象存储到一个对象数组时抛出的异常。 |
| ClassCastException | 当试图将对象强制转换为不是实例的子类时，抛出该异常。 |
| IllegalArgumentException | 抛出的异常表明向方法传递了一个不合法或不正确的参数。 |
| IllegalMonitorStateException | 抛出的异常表明某一线程已经试图等待对象的监视器，或者试图通知其他正在等待对象的监视器而本身没有指定监视器的线程。 |
| IllegalStateException | 在非法或不适当的时间调用方法时产生的信号。换句话说，即 Java 环境或 Java 应用程序没有处于请求操作所要求的适当状态下。 |
| IllegalThreadStateException | 线程没有处于请求操作所要求的适当状态时抛出的异常。 |
| IndexOutOfBoundsException | 指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。 |
| NegativeArraySizeException | 如果应用程序试图创建大小为负的数组，则抛出该异常。 |
| NullPointerException | 当应用程序试图在需要对象的地方使用 null 时，抛出该异常 |
| NumberFormatException | 当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。 |
| SecurityException | 由安全管理器抛出的异常，指示存在安全侵犯。 |
| StringIndexOutOfBoundsException | 此异常由 String 方法抛出，指示索引或者为负，或者超出字符串的大小。 |
| UnsupportedOperationException | 当不支持请求的操作时，抛出该异常。 |

## API
| **public String getMessage()**
返回关于发生的异常的详细信息。这个消息在Throwable 类的构造函数中初始化了。 |
| --- |
| **public Throwable getCause()**
返回一个 Throwable 对象代表异常原因。 |
| **public String toString()**
返回此 Throwable 的简短描述。 |
| **public void printStackTrace()**
将此 Throwable 及其回溯打印到标准错误流。。 |
| **public StackTraceElement [] getStackTrace()**
返回一个包含堆栈层次的数组。下标为0的元素代表栈顶，最后一个元素代表方法调用堆栈的栈底。 |
| **public Throwable fillInStackTrace()**
用当前的调用栈层次填充Throwable 对象栈层次，添加到栈层次任何先前信息中。 |

## 关键字
- **try** – 用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
- **catch** – 用于捕获异常。catch用来捕获try语句块中发生的异常。
- **finally** – finally语句块总是会被执行。它主要用于回收在try块里打开的物力资源(如数据库连接、网络连接和磁盘文件)。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。
- **throw** – 用于抛出异常。
- **throws** – 用在方法签名中，用于声明该方法可能抛出的异常。

## 抛出异常
抛出异常有两种方式：**方法头上声明，方法体中抛出。**

异常抛出的规则：
- 不可查异常可以不抛出异常，编译仍能顺利通过，但在运行时会被系统抛出。
- 如果出现可查异常，要么用`try-catch`捕获处理，要么用`throws`声明抛出，否则编译错误、
- 当方法抛出了异常，该方法的调用者必须处理或者再次抛出异常。
- 调用方法必须遵循任何可查异常的处理和声明规则。若覆盖一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或子类。

### 声明
使用`throws`关键字在方法头声明方法抛出的异常。
```java
public static void method() throws IOException, FileNotFoundException{
    //something statements
}
```

### 抛出
如果代码可能会引发某种错误，可以创建一个合适的异常类实例并抛出它。
```java
public static double method(int value) {
    if(value == 0) {
        throw new ArithmeticException("参数不能为0"); //抛出一个运行时异常
    }
    return 5.0 / value;
}
```

有时会从 catch 中抛出一个异常，目的是为了改变异常的类型。多用于在多系统集成时，当某个子系统故障，异常类型可能有多种，可以用统一的异常类型向外暴露，不需暴露太多内部异常细节。
```java
private static void readFile(String filePath) throws MyException {    
    try {
        // code
    } catch (IOException e) {
        MyException ex = new MyException("read file failed.");
        ex.initCause(e);
        throw ex;
    }
}
```

## 捕获异常
捕获程序中可能出现的异常进行处理。

异常捕获原则：
- 尽量添加`finally`语句块去释放占用的资源。
- 在多重`catch`块后面，可以加一个`catch(Exception)`来处理可能会被遗漏的异常。
- 对于不确定的代码，加上`try-catch`可以处理潜在的异常。
- `throws`抛出异常并没有被处理，类似主函数的方法最好不要抛出异常。

### try-catch-finally
`try-catch`语句块中可以捕获多个异常类型，并对不同类型的异常做出不同的处理。
同一个 `catch` 也可以捕获多种类型异常，用`|`隔开。
`finally`块是可选的。
```java
private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException | UnknownHostException e) {
        // handle FileNotFoundException
    } catch (IOException e){
        // handle IOException
    } finally {
    // 总会被执行的代码
}
```

执行顺序：
1. 当`try`没有捕获到异常时：跳过catch语句块，执行finally语句块和其后的语句。
2. 当`try`捕获到异常且 `catch`语句块里有处理此异常的情况：跳到`catch`语句块找到与之对应的处理程序，其他的`catch`语句块将不会被执行，最后执行`finally`语句块后的语句。
3. 当`try`捕获到异常且 `catch`语句块里没有处理此异常的情况：此异常将会抛给JVM处理，`finally`语句块里的语句还是会被执行。

```java
private static void readFile(String filePath) throws MyException {
    File file = new File(filePath);
    String result;
    BufferedReader reader = null;
    try {
        reader = new BufferedReader(new FileReader(file));
        while((result = reader.readLine())!=null) {
            System.out.println(result);
        }
    } catch (IOException e) {
        System.out.println("readFile method catch block.");
        MyException ex = new MyException("read file failed.");
        ex.initCause(e);
        throw ex;
    } finally {
        System.out.println("readFile method finally block.");
        if (null != reader) {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### try-finally
`try-finally`，因为`finally`块中的语句始终会被执行，所以可用在不需要捕获异常的代码，保证资源在使用后被关闭。
常用场景有：关闭IO流，Lock释放，DB连接关闭 等。

`finally`在特殊情况下导致的不执行：
- 在前面的代码中用了`System.exit()`退出程序。
- `finally`语句块中发生了异常。
- 程序所在的线程死亡。
- 关闭CPU。

```java
//以Lock加锁为例，演示try-finally
ReentrantLock lock = new ReentrantLock();
try {
    //需要加锁的代码
} finally {
    lock.unlock(); //保证锁一定被释放
}
```

### try-with-resource
因为`finally` 中的`close`方法也可能抛出 `IOException`, 从而覆盖原始异常。JAVA 7 提供了更优雅的方式实现资源的自动释放，可以自动释放的资源要求继承`AutoCloseable`类。
`try-with-resources`语句使用`try()`声明资源，资源会自动释放。声明多个资源使用分号分隔。

`try`代码块退出时自动调用`close()`方法，和`finally`块不同的是，若`close()`抛出异常会被抑制，抛出的仍然为原始异常。被抑制的异常由`addSusppressed()`方法添加到原来的异常，如果想获取被抑制的异常列表，可以调用`getSuppressed`方法获取。
```java
// Scanner类实现的Closeable接口实继承了AutoCloseable类
public final class Scanner implements Iterator<String>, Closeable {// ...}
public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}

private static void tryWithResourceTest(){
    try (Scanner scanner = new Scanner(new FileInputStream("c:/abc"),"UTF-8")){
        // code
    } catch (IOException e){
        // handle exception
    }
}
```
## 自定义异常
在业务中常需要自定义异常，可以防止向外暴露过多系统的细节。

创建的异常必须满足以下条件：
- 必须是`Throwable`的子类。
- 如果是检查性异常，需要继承`Exception`类。
- 如果是运行时异常类，需要继承`RuntimeException`类。
- 包含两个构造函数，无参构造和带有详细描述信息的构造函数（`Throwable` 的 `toString()` 方法会打印详细信息，调试时很有用）

```java
public class MyException extends Exception {
    public MyException(){ }
    public MyException(String msg){
        super(msg);
    }
    // ...
}
```

## 常用异常

### RuntimeException
- java.lang.ArrayIndexOutOfBoundsException 数组索引越界异常。当对数组的索引值为负数或大于等于数组大小时抛出。
- java.lang.ArithmeticException 算术条件异常。譬如：整数除零等。
- java.lang.NullPointerException 空指针异常。当应用试图在要求使用对象的地方使用了null时，抛出该异常。譬如：调用null对象的实例方法、访问null对象的属性、计算null对象的长度、使用throw语句抛出null等等
- java.lang.ClassNotFoundException 找不到类异常。当应用试图根据字符串形式的类名构造类，而在遍历CLASSPAH之后找不到对应名称的class文件时，抛出该异常。
- java.lang.NegativeArraySizeException 数组长度为负异常
- java.lang.ArrayStoreException 数组中包含不兼容的值抛出的异常
- java.lang.SecurityException 安全性异常
- java.lang.IllegalArgumentException 非法参数异常

### IOException
- IOException：操作输入流和输出流时可能出现的异常。
- EOFException 文件已结束异常
- FileNotFoundException 文件未找到异常

### 其他
- ClassCastException 类型转换异常类
- ArrayStoreException 数组中包含不兼容的值抛出的异常
- SQLException 操作数据库异常类
- NoSuchFieldException 字段未找到异常
- NoSuchMethodException 方法未找到抛出的异常
- NumberFormatException 字符串转换为数字抛出的异常
- StringIndexOutOfBoundsException 字符串索引超出范围抛出的异常
- IllegalAccessException 不允许访问某类异常
- InstantiationException 当应用程序试图使用Class类中的newInstance()方法创建一个类的实例，而指定的类对象无法被实例化时，抛出该异常

## 异常实践经验
抛出或捕获异常时，大部分是为了改善代码的可读性或者 API 的可用性。异常不仅是错误控制机制，也是通信媒介。为了更好的合作，团队必须制定出一个最佳实践规则。

### 非必要不使用
异常只应该被用于不正常的条件，它们永远不应该被用于正常的控制流。
> 《阿里手册》中：【强制】Java 类库中定义的可以通过预检查方式规避的RuntimeException异常不应该通过catch 的方式来处理，比如：NullPointerException，IndexOutOfBoundsException等等。

异常机制设计初衷是用于不正常的情况，所以很少的JVM对它们的性能进行优化。所以，创建、抛出、捕获异常的开销是昂贵的。把代码放在try-catch中返回阻止了JVM实现本来可能要执行的某些特定的优化。
```java
// 建议避免异常
if (obj != null)

// 而不建议捕捉这样的异常
try {} 
catch (NullPointerException e) {}
catch (IndexOutOfBoundsException e) {}
```

### 释放资源方式
把清理工作的代码放到 `finally` 里，或者使用 `try-with-resource` 特性。
```java
public void closeResourceInFinally() {
    FileInputStream inputStream = null;
    try {
        File file = new File("./tmp.txt");
        inputStream = new FileInputStream(file);
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                log.error(e);
            }
        }
    }
}
```
```java
/*
如果资源实现了 AutoCloseable 接口，可以使用这个语法。
大多数的 Java 标准资源都继承了这个接口。
在 try 子句中打开资源，资源会在 try 代码块执行后或异常处理后自动关闭。
*/
public void automaticallyCloseResource() {
    File file = new File("./tmp.txt");
    try (FileInputStream inputStream = new FileInputStream(file);) {
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```

### 尽量使用标准异常
代码重用是值得提倡的，异常也不例外。重用现有异常的可读性更好，因为不会充斥着不熟悉的异常。且异常类越少，内存占用越小，转载这些类的时间开销也越小。

Java标准异常中常用的异常：
| IllegalArgumentException | 参数的值不合适 |
| --- | --- |
| IllegalStateException | 参数的状态不合适 |
| NullPointerException | 在null被禁止的情况下参数值为null |
| IndexOutOfBoundsException | 下标越界 |
| ConcurrentModificationException | 在禁止并发修改的情况下，对象检测到并发修改 |
| UnsupportedOperationException | 对象不支持客户请求的方法 |

### 文档说明异常
在 Javadoc 添加 `@throws` 声明，并且描述抛出异常的场景。
在抛出自定义异常时，需要尽可能精确地描述问题相关信息，这样无论是打印到日志中还是在监控工具中，都更容易被阅读，更好地定位具体错误、严重程度等。

### 具体异常优先捕获
总是优先捕获最具体的异常类，并将不太具体的 `catch` 块添加到列表的末尾。如果首先捕获了更不具体的异常，则永远捕捉不到具体的异常。

### 不捕获Throwable
`Throwable` 是所有异常和错误的超类。可以在 catch 子句中使用它，但永远不应该这样做！除非你确定自己处于一种特殊的情况下能够处理错误。
它不仅会捕获异常，也捕获所有的错误。JVM 抛出错误，指出不应该由应用程序处理的严重问题。 典型的例子是 `OutOfMemoryError` 或者 `StackOverflowError` 。两者都是由应用程序控制之外的情况引起的，无法处理。

### 不忽略异常
catch块没有做任何处理或者记录日志，自信不会抛出异常。但现实是经常会出现无法预料的异常，或者无法确定这里的代码未来是不是会改动(删除了阻止异常抛出的代码)，而此时由于异常被捕获，使得无法拿到足够的错误信息来定位问题。合理的做法是至少要记录异常的信息。

```java
public void logAnException() {
    try {
        // do something
    } catch (NumberFormatException e) {
        log.error("This should never happen: " + e); // see this line
    }
}
```
### 不又记录又抛出
捕获异常、记录日志并再次抛出的逻辑看似合理。但这会给同一个异常输出多条日志。

### 处理异常才捕获
仅当想要处理异常时才去捕获，否则只需要在方法签名中声明让调用者去处理。

### 不抛弃原始异常
捕获标准异常并包装为自定义异常是一个很常见的做法。可以添加更为具体的异常信息并能够做针对的异常处理。 在这样做时，确保将原始异常设置为原因。
`Exception` 类提供了特殊的构造函数方法，接收 Throwable 作为参数。否则将会丢失堆栈跟踪和原始异常的消息，这将会使分析导致异常的异常事件变得困难。

### 异常不控制流程
不应该使用异常控制应用的执行流程，应该使用`if`语句时使用异常处理，这会严重影响应用的性能。

### finally块中不使用return。
`try`块中的`return`语句执行成功后，并不会马上返回，而是继续执行`finally`块中的语句，如果此处存在`return`语句，则在此直接返回，无情丢弃掉`try`块中的返回点。
```java
private int x = 0;
public int checkReturn() {
    try {
        // x等于1，此处不返回
        return ++x;
    } finally {
        // 返回的结果是2
        return ++x;
    }
}
```

## JVM异常处理机制
JVM处理异常的机制，需要提及Exception Table，也称为异常表。

借助`javap`用来拆解class文件的工具，和`javac`一样由JDK提供。先使用javac编译然后使用javap来分析这段代码。看到 Exception table 就是要研究的异常表。
```java
//javap -c Main
 public static void simpleTryCatch();
    Code:
       0: invokestatic  #3                  // Method testNPE:()V
       3: goto          11
       6: astore_0
       7: aload_0
       8: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
      11: return
    Exception table:
       from    to  target type
           0     3     6   Class java/lang/Exception
```
异常表中包含了一个或多个异常处理者(Exception Handler)的信息，这些信息包括：

- **from** 可能发生异常的起始点
- **to** 可能发生异常的结束点
- **target** 上述from和to之前发生异常后的异常处理者的位置
- **type** 异常处理者处理的异常的类信息

---

异常表用在异常发生的时候，当一个异常发生时

1. JVM会在当前出现异常的方法中，查找异常表，是否有合适的处理者来处理。
2. 如果当前方法异常表不为空，并且异常符合处理者的from和to节点，并且type也匹配，则JVM调用位于target的调用者来处理。
3. 如果上一条未找到合理的处理者，则继续查找异常表中的剩余条目。
4. 如果当前方法的异常表无法处理，则向上查找（弹栈处理）刚刚调用该方法的调用处，并重复上面的操作。
5. 如果所有的栈帧被弹出，仍然没有处理，则抛给当前的Thread，Thread则会终止。
6. 如果当前Thread为最后一个非守护线程，且未处理异常，则会导致JVM终止运行。

以上就是JVM处理异常的一些机制。

### try catch -finally
try-catch-finally结合使用的异常表。
```java
public static void simpleTryCatchFinally();
    Code:
       0: invokestatic  #3                  // Method testNPE:()V
       3: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
       6: ldc           #7                  // String Finally
       8: invokevirtual #8    // Method java/io/PrintStream.println(Ljava/lang/String;)V
      11: goto          41
      14: astore_0
      15: aload_0
      16: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
      19: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      22: ldc           #7                  // String Finally
      24: invokevirtual #8    // Method java/io/PrintStream.println(Ljava/lang/String;)V
      27: goto          41
      30: astore_1
      31: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      34: ldc           #7                  // String Finally
      36: invokevirtual #8    // Method java/io/PrintStream.println(Ljava/lang/String;)V
      39: aload_1
      40: athrow
      41: return
    Exception table:
       from    to  target type
           0     3    14   Class java/lang/Exception
           0     3    30   any
          14    19    30   any
  
```
这次异常表中，有三条数据，而仅仅捕获了一个Exception, 异常表的后两个item的type为any; 上面的三条异常表item的意思为:

- 如果0到3之间，发生了Exception类型的异常，调用14位置的异常处理者。
- 如果0到3之间，无论发生什么异常，都调用30位置的处理者
- 如果14到19之间（即catch部分），不论发生什么异常，都调用30位置的处理者。

再次分析上面的Java代码，**finally里面的部分已经被提取到了try部分和catch部分**。再次调一下代码：
```java
public static void simpleTryCatchFinally();
    Code:
      //try 部分提取finally代码，如果没有异常发生，则执行输出finally操作，直至goto到41位置，执行返回操作。  

       0: invokestatic  #3                  // Method testNPE:()V
       3: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
       6: ldc           #7                  // String Finally
       8: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      11: goto          41

      //catch部分提取finally代码，如果没有异常发生，则执行输出finally操作，直至执行got到41位置，执行返回操作。
      14: astore_0
      15: aload_0
      16: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
      19: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      22: ldc           #7                  // String Finally
      24: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      27: goto          41
      //finally部分的代码如果被调用，有可能是try部分，也有可能是catch部分发生异常。
      30: astore_1
      31: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      34: ldc           #7                  // String Finally
      36: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      39: aload_1
      40: athrow     //如果异常没有被catch捕获，而是到了这里，执行完finally的语句后，仍然要把这个异常抛出去，传递给调用处。
      41: return
```
**Catch先后顺序的问题**
在代码中的catch的顺序决定了异常处理者在异常表的位置，所以，越是具体的异常要先处理，否则就会导致编译失败，如果先捕获Throwable后捕获Exception，会导致后面的catch永远无法被执行。

### Return 和finally的问题
一个相对比较极端的问题，既有return，又有finally，那么finally会不会执行。
```java
public static String tryCatchReturn() {
   try {
       testNPE();
       return  "OK";
   } catch (Exception e) {
       return "ERROR";
   } finally {
       System.out.println("tryCatchReturn");
   }
}
```
finally会执行，使用javap分析一下为什么finally会执行。
```java
public static java.lang.String tryCatchReturn();
    Code:
       0: invokestatic  #3                  // Method testNPE:()V
       3: ldc           #6                  // String OK
       5: astore_0
       6: getstatic     #7          // Field java/lang/System.out:Ljava/io/PrintStream;
       9: ldc           #8                  // String tryCatchReturn
      11: invokevirtual #9   // Method java/io/PrintStream.println(Ljava/lang/String;)V
      14: aload_0
      15: areturn       //返回OK字符串，areturn意思为return a reference from a method
      16: astore_0
      17: ldc           #10                 // String ERROR
      19: astore_1
      20: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
      23: ldc           #8                  // String tryCatchReturn
      25: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      28: aload_1
      29: areturn  //返回ERROR字符串
      30: astore_2
      31: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
      34: ldc           #8                  // String tryCatchReturn
      36: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      39: aload_2
      40: athrow  如果catch有未处理的异常，抛出去。
```

## 异常为什么耗时
使用异常慢，慢在哪里？有多慢？下面的测试用例简单的测试了建立对象、建立异常对象、抛出并接住异常对象三者的耗时对比：
```java
public class ExceptionTest {  
  
    private int testTimes;  
  
    public ExceptionTest(int testTimes) {  
        this.testTimes = testTimes;  
    }  
    
    /*
    运行结果
    建立对象：575817  
建立异常对象：9589080  
建立、抛出并接住异常对象：47394475  
    */
  
    public void newObject() {  
        long l = System.nanoTime();  
        for (int i = 0; i < testTimes; i++) {  
            new Object();  
        }  
        System.out.println("建立对象：" + (System.nanoTime() - l));  
    }  
  
    public void newException() {  
        long l = System.nanoTime();  
        for (int i = 0; i < testTimes; i++) {  
            new Exception();  
        }  
        System.out.println("建立异常对象：" + (System.nanoTime() - l));  
    }  
  
    public void catchException() {  
        long l = System.nanoTime();  
        for (int i = 0; i < testTimes; i++) {  
            try {  
                throw new Exception();  
            } catch (Exception e) {  
            }  
        }  
        System.out.println("建立、抛出并接住异常对象：" + (System.nanoTime() - l));  
    }  
  
    public static void main(String[] args) {  
        ExceptionTest test = new ExceptionTest(10000);  
        test.newObject();  
        test.newException();  
        test.catchException();  
    }  
}  

/*
运行结果
建立对象：575817  
建立异常对象：9589080  
建立、抛出并接住异常对象：47394475  
*/
```
建立一个异常对象，是建立一个普通Object耗时的约20倍（实际上差距会比这个数字更大一些，因为循环也占用了时间，追求精确可以再测一下空循环的耗时然后在对比前减掉这部分），而抛出、接住一个异常对象，所花费时间大约是建立异常对象的4倍。

### 异常发生瞬间
RednaxelaFX：用字节码来解释性能问题很抱歉也是比较不靠谱的。字节码用于解释“语义问题”很靠谱，但看字节码是看不出性能问题的——超过它的抽象层次了
	IcyFenix：同意RednaxelaFX的观点，但本节中的字节码分解本就不涉及性能，仅想表达“当异常发生那一刹那”时会发生什么事情（准确的说，还要限定为是解释方式执行），这里提及的20%、80%时间是基于前一点测试的结果。
 	把`catchException()`方法中循环和时间统计的代码去掉，使得代码变得纯粹一些：
```java
public void catchException() {
	try {
		throw new Exception();
	} catch (Exception e) {
	}
}
```
然后使用javap -verbose命令输出它的字节码，结果如下：
```java
public void catchException();
  Code:
   Stack=2, Locals=2, Args_size=1
   0:   new     #58; //class java/lang/Exception
   3:   dup
   4:   invokespecial   #60; //Method java/lang/Exception."<init>":()V
   7:   athrow
   8:   astore_1
   9:   return
  Exception table:
   from   to  target type
     0     8     8   Class java/lang/Exception
```

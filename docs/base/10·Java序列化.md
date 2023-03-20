# 10·Java序列化

- [10·Java序列化](#10java序列化)
  - [什么是序列化](#什么是序列化)
  - [serialVersionUID](#serialversionuid)
  - [transient](#transient)
  - [单例模式增强](#单例模式增强)
  - [fastjson](#fastjson)

## 什么是序列化
序列化是希望对一个Java对象变成字节序列，方便持久化存储到磁盘，避免程序运行结束后对象就从内存里消失，变换成字节序列也更便于网络运输和传播。想要将一个对象序列化，只需要实现Serializable接口即可，但这是一个空接口，只是作为一个标识。真正序列化的操作不需要它来完成。
反序列化是把字节序列恢复为原先的Java对象，反序列化也是一个创建对象的方式。

## serialVersionUID
`serialVersionUID`是序列化前后的唯一标识符序列化ID。是序列化和反序列化过程中的“暗号”，在反序列化时，JVM会把字节流中的序列号ID和被序列化类中的序列号ID做比对，只有两者一致，才能重新反序列化，否则就会报异常来终止反序列化的过程。
如果没有显式定义过`serialVersionUID`，那编译器会自动声明一个，一旦更改了类的结构或者信息，则类的`serialVersionUID`也会跟着变化。为了`serialVersionUID`的确定性，建议需要序列化的类都显式地声明一个`serialVersionUID`明确值。

## transient
`transient`关键字修饰的字段不会被序列化。所以不希望某个字段被序列化（隐私值），在序列化对象时，字段会设置为`NULL`。
此外被`static`修饰的字段也是不会被序列化的。

## 单例模式增强
**可序列化的单例类有可能并不单例！**解决办法是在单例类中手写`readResolve()`函数，直接返回单例对象来规避。这样当反序列化从流中读取对象时，`readResolve()`会被调用，用其中返回的对象替代反序列化新建的对象。

## fastjson
`JSON.toJSONString()`：序列化，将Java对象序列化为Json格式字符串。
`JSON.parseObject()`：反序列化，将Json格式字符串转换为Java对象，生成对象的一种方式。
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastJson.JSONObject;

public Object insert(Map<String,Object> param,User user,BusinessExecuteService b){
    // 将Map对象转换为Json格式字符串
    String str = JSON.toJSONString(param);
    // 将Json格式字符串反序列化为对象
    Query query = JSON.parseObject(str,Query.class);
}
```
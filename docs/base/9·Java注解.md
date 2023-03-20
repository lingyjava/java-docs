# 9·Java注解

- [9·Java注解](#9java注解)
  - [什么是注解](#什么是注解)
  - [内置注解](#内置注解)
  - [元注解](#元注解)
  - [自定义注解](#自定义注解)

## 什么是注解
注解（Annotation）不是程序本身，可以对程序做出解释。**注解可以被其他程序读取。**

## 内置注解
| @Override | 标记重写的方法 |
| --- | --- |
| @Deprecated | 方法或者类被弃用，有其他方法或者类可以代替使用，以后不再维护这个方法或类，就可以加上此注解，方法名会增加一条删除线 |
| @SuppressWarnings() | 镇压警告，需要参数 |

## 元注解
元注解（meta-annotation）负责注解其他注解的注解。

Java中定义了4个标准的元注解类型，用来提供对其他注解类型作说明。
| @Target | 描述注解的使用范围 |
| --- | --- |
| @Retention | 表示需要在什么级别保存该注释信息，描述注解的生命周期(SOURCE<CLASS<RUNTIME) |
| @Documented | 说明该注解将被包含在javadoc中 |
| @Inherited | 说明子类可以继承父类中的该注解 |

## 自定义注解
使用`@interface`来声明一个注解`public @interface 注解名{}` ，其自动继承`java.lang.annotation.Annotation`接口。

定义规则：
1.  每个方法实际上是声明一个配置参数。 
2.  方法的名称就是参数的名称。 
3.  返回值类型就是参数的类型（返回值只能是基本类型，Class，String，enum）。
4.  通过`default`声明参数的默认值。 
5.  如果只有一个参数，一般参数名为`value`。 
6.  注解元素必须要有值，定义注解元素时，经常使用空字符串0作为默认值。

```java
//自定义一个注解格式
@Target(value = ElementType.METHOD)//在方法中生效
@Retention(value = RetentionPolicy.RUNTIME)//表示在运行时也生效(任何时生效)
@Documented
@Inherited
public @interface MyFirstAnnotation{
    //注解的参数 ： 参数类型 + 参数名();
    String name() default "";
    int age() default 0;
    int id() default -1;//默认-1，代表不存在
}

//使用时必须需要提供无默认值的参数
@MyFirstAnnotation(name = "lee",age = 18,id = 10086)
public void test1(){}
```

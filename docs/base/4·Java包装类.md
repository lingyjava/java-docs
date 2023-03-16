# 4·Java包装类

- [4·Java包装类](#4java包装类)
  - [包装类是什么](#包装类是什么)
  - [基本数据类型包装类](#基本数据类型包装类)
  - [常量池技术](#常量池技术)
  - [装拆箱](#装拆箱)
  - [Integr API](#integr-api)
  - [Character API](#character-api)

## 包装类是什么
包装类是基于Java中8大基本数据类型的封装，值允许为NULL（默认值为NULL），并提供了实用API。

**包装类之间的比较必须使用equals方法。**

## 基本数据类型包装类
| long | Long |
| --- | --- |
| int | Integer |
| short | Short |
| byte | Byte |
| double | Double |
| float | Float |
| boolean | Boolean |
| char | Character |

## 常量池技术
大部分包装类默认创建了部分常用数值。

- 整型包装类缓存值（-127 ~ 128）。
- `Character`缓存值（0 ~ 127）。
- `Boolean`缓存值（True 、False）。
- 浮点型包装类没有使用常量池技术。

## 装拆箱
装拆箱是指基本数据类型与对于的包装类型互相转换的过程。大部分情况下，装拆箱可以自动进行。
避免非必要的装拆箱，频繁装拆箱影响系统性能。

- 装箱：将基本数据类型转换为对应的包装类型。
- 拆箱：将包装类型转换为对应的基本数据类型。

```java
Integer i = 40; // 装箱 等价于 Integer i = Integer.valueOf(40);
int j = i; // 拆箱 等价于 int j = i.intValue();
```

## Integr API
| compareTo() | 比较两个对象的大小
1（前面大）
0（相等）
-1（后面大） |
| --- | --- |
| parseInt() | 将字符串类型的参数转换为带符号的十进制整数 |
| toBinaryString() | 将十进制转换为二进制 |
| toOctalString() | 将十进制转换为八进制 |
| toHexString() | 将十进制转换为十六进制 |

## Character API
| isDigit() | 是否是数字 |
| --- | --- |
| isLetter() | 是否是字母 |
| isLowerCase() | 是否是小写字母 |
| reverseBytes() | 反转字节 |



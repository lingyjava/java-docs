# 6·Bigdecimal

- [6·Bigdecimal](#6bigdecimal)
  - [Bigdecimal对象是什么](#bigdecimal对象是什么)
  - [构造方法](#构造方法)
  - [比较](#比较)
  - [API](#api)
  - [舍入模式](#舍入模式)
  - [使用示例](#使用示例)
  - [常量](#常量)
  - [格式化](#格式化)

## Bigdecimal对象是什么
BigDecimal可以作为要求浮点计算精度时的数据类型，比如金额。BigDecimal是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象，由于创建对象会引起开销，因此不适合于大量的数学运算。

## 构造方法
BigDecimal够造方法的参数类型有4种，其中的两个用`BigInteger`构造，另一个是用`double`构造，还有一个使用`String`构造。
**避免使用**`double`**构造**，有些数字用`double`无法精确表示，传给`BigDecimal`构造方法时就已经不精确了。应该用`String`构造`BigDecimal`。

```java
new BigDecimal(0.1);// 得到的值是0.1000000000000000055511151231257827021181583404541015625
new BigDecimal("0.1");// 得到的值是0.1
```

## 比较
在从数值上比较两个BigDecimal值时，避免使用`equals()`比较，应该使用`compareTo()`。
```java
Bigdecimal b1 = new BigDecimal("0.1");
Bigdecimal b2 = new BigDecimal("0.1");
Bigdecimal b3 = new BigDecimal("0.10");
b1.equals(b2); // true
b1.equals(b3); // false
b1.compareTo(b2); //true
b1.compareTo(b3); //true
```
## API
| add() | 相加 |
| --- | --- |
| subtract() | 相减 |
| multiply() | 相乘 |
| divide() | 相除 |
| doubleValue() | 将Bigdecimal对象转换为double类型 |

## 舍入模式
任意精度的小数运算仍不能表示精确结果。经常可能产生无限循环的小数。在进行除法运算时，BigDecimal可以显式地控制舍入模式。

`RoundingMode`舍入模式
`DOWN`：零向舍入(舍弃后边所有，不向前进1)
`UP`：舍入去零，零不舍入。
`HELF_DOWN`：遵循四舍五入规则，大于5向前一位进1
`HELF_UP`：遵循四舍五入规则，大于等于5向前一位进1

## 使用示例
```java
//创建四舍五入的配置对象（四舍五入保留3位有效数字）
MathContext mc1 = new MathContext(3,RoudingMode.HELF_UP);
b1.setScale(3, RoundingMode.HALF_UP);//保留3位小数
BigDecimal b3 = new BigDecimal("3.1415926");
// 使用自定义配置对象
System.out.println(b3.round(mc1))//3.14
```

## 常量
`Bigdecimal`提供了常用的数值常量，不需要额外创建对象。

| Bigdecimal.ZERO | 0 |
| --- | --- |

## 格式化
`DecimalFormat`对象可以对`Bigdecimal`数据类型进行格式化的对象。

```java
// 声明格式化对象
DecimalFormat df1 = new DecimalFormat("#,###.000");
System.out.println(df1.format(1234.527));
// 结果：1,234.527

// 声明格式化对象
DecimalFormat df2 = new DecimalFormat("#.00");
System.out.println(df2.format(1234.527));
// 结果：1234.53（发生四舍五入）

// 声明格式化对象
DecimalFormat df3 = new DecimalFormat("##.00%");
System.out.println(df3.format(0.527));
// 结果：52.70%（百分比格式，后面不足2位的用0补齐）

// 切换为百分比格式，声明数值格式化对象
NumberFormat nf = NumberFormat.getPercentInstance();
// 保留到小数点后几位
nf.setMinimumFractionDigits(3);    
System.out.println(nf.format(0.527));
// 结果：52.700%

System.out.println(String.format("%02d", 5));
// 结果：05
System.out.println(String.format("%.3f", 3.14159));
// 结果：3.142,%. 表示小数点前任意位数, 2 表示两位小数, f 表示浮点型

NumberFormat format = NumberFormat.getNumberInstance();
format.setMinimumFractionDigits(3);//设置小数部分允许的最小位数
format.setMaximumFractionDigits(5);//设置小数部分允许的最大位数
format.setMaximumIntegerDigits(10);//设置整数部分允许的最大位数。
format.setMinimumIntegerDigits(0);//设置整数部分允许的最小位数
System.out.println(format.format(2132323213.23266666666)); 2,132,323,213.23267
```

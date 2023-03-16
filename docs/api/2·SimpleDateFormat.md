# SimpleDateFormat

- [SimpleDateFormat](#simpledateformat)
  - [SimpleDateFormat对象是什么](#simpledateformat对象是什么)
  - [构造方法](#构造方法)
  - [解析时间](#解析时间)
  - [为什么线程不安全](#为什么线程不安全)
  - [解决线程不安全](#解决线程不安全)
    - [创建新实例](#创建新实例)
    - [同步锁](#同步锁)
    - [ThreadLocal](#threadlocal)
    - [其他类库](#其他类库)
  - [参考](#参考)

## SimpleDateFormat对象是什么
`java.text.SimpleDateFormat`

时间格式化工具类，线程不安全。

## 构造方法
构造方法中传入解析方式`pattern`。

| **解析方式** |  |
| --- | --- |
| Date and Time Pattern | Result |
| "yyyy.MM.dd G 'at' HH:mm:ss z" | 2001.07.04 AD at 12:08:56 PDT |
| "EEE, MMM d, ''yy" | Wed, Jul 4, '01 |
| "h:mm a" | 12:08 PM |
| "hh 'o''clock' a, zzzz" | 12 o'clock PM, Pacific Daylight Time |
| "K:mm a, z" | 0:08 PM, PDT |
| "yyyyy.MMMMM.dd GGG hh:mm aaa" | 02001.July.04 AD 12:08 PM |
| "EEE, d MMM yyyy HH:mm:ss Z" | Wed, 4 Jul 2001 12:08:56 -0700 |
| "yyMMddHHmmssZ" | 010704120856-0700 |
| "yyyy-MM-dd'T'HH:mm:ss.SSSZ" | 2001-07-04T12:08:56.235-0700 |
| "yyyy-MM-dd'T'HH:mm:ss.SSSXXX" | 2001-07-04T12:08:56.235-07:00 |
| "YYYY-'W'ww-u" | 2001-W27-3 |
| "yyyy-MM-dd HH:mm:ss" | 2022-12-01 15:04:59 |

## 解析时间
| format() | 时间解析为字符串 |
| --- | --- |
| parse() | 字符串解析为时间 |

## 为什么线程不安全
`SimpleDateFormat`中的日期格式不是同步的。推荐（建议）为每个线程创建独立的格式实例。如果多个线程同时访问一个格式，则它必须保持外部同步。
 	`SimpleDateFormat`继承了`DateFormat`，在`DateFormat`中定义了一个`protected`属性的`Calendar`类的对象。因为`Calendar`类的概念复杂，牵扯到时区与本地化等等，Jdk的实现中使用了成员变量来传递参数，这就造成在多线程的时候会出现错误。
在一定负载情况下时，会出现不同的情况，比如转化的时间不正确、报错、线程被挂死等。

## 解决线程不安全

### 创建新实例
在每次使用时创建新的实例。在需要对大量时间进行解析转换时，每次都创建一个新的实例可能对性能造成影响。
```java
package com.peidasoft.dateformat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateUtil {

    public static  String formatDate(Date date)throws ParseException{
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return sdf.format(date);
    }

    public static Date parse(String strDate) throws ParseException{
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return sdf.parse(strDate);
    }
}
```

### 同步锁
使用同步锁。想要调用此方法的线程就要block，多线程并发量大的时候会对性能有一定的影响。
```java
package com.peidasoft.dateformat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateSyncUtil {

    private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
      
    public static String formatDate(Date date)throws ParseException{
        synchronized(sdf){
            return sdf.format(date);
        }  
    }
    
    public static Date parse(String strDate) throws ParseException{
        synchronized(sdf){
            return sdf.parse(strDate);
        }
    } 
}
```

### ThreadLocal
使用`ThreadLocal`, 将共享变量变为独享，线程独享肯定能比方法独享在并发环境中能减少不少创建对象的开销。如果对性能要求比较高的情况下，一般推荐使用这种方法。
```java
package com.peidasoft.dateformat;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class ConcurrentDateUtil {

    private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>() {
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static Date parse(String dateStr) throws ParseException {
        return threadLocal.get().parse(dateStr);
    }

    public static String format(Date date) {
        return threadLocal.get().format(date);
    }
}
```
```java
package com.peidasoft.dateformat;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class ThreadLocalDateUtil {
    private static final String date_format = "yyyy-MM-dd HH:mm:ss";
    private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>(); 
	public static DateFormat getDateFormat() {  
        DateFormat df = threadLocal.get();  
        if(df == null){  
            df = new SimpleDateFormat(date_format);  
            threadLocal.set(df);  
        }  
        return df;  
    }  

    public static String formatDate(Date date) throws ParseException {
        return getDateFormat().format(date);
    }

    public static Date parse(String strDate) throws ParseException {
        return getDateFormat().parse(strDate);
    }   
}
```

### 其他类库
1. Apache commons 里的`FastDateFormat`，宣称是既快又线程安全的SimpleDateFormat, 可惜它只能对日期进行format, 不能对日期串进行解析。
2. 使用Joda-Time类库来处理时间相关问题。

## 参考
- [深入理解Java：SimpleDateFormat安全的时间格式化](https://developer.aliyun.com/article/651819)
# 4·String

- [4·String](#4string)
  - [String对象是什么](#string对象是什么)
  - [不可变性](#不可变性)
  - [charAt()](#charat)
    - [查找字符](#查找字符)
  - [substring](#substring)
    - [使用示例](#使用示例)
  - [concat](#concat)
    - [使用示例](#使用示例-1)
  - [split](#split)
    - [使用示例](#使用示例-2)
  - [join](#join)
    - [使用示例](#使用示例-3)
  - [repeat()](#repeat)
    - [使用示例](#使用示例-4)
  - [isEmpty](#isempty)
  - [StringBuilder \& StringBuffer](#stringbuilder--stringbuffer)
    - [使用示例](#使用示例-5)
  - [StringAPI](#stringapi)
  - [统计每个字符出现次数](#统计每个字符出现次数)

## String对象是什么
Java没有内置字符串类型，而在标准Java类库中提供了一个预定义类对象（String），从概念上看，Java字符串就是Unicode字符序列。字符串下标从0开始。

## 不可变性
Java文档中将String类对象称为不可变的，每次修改字符串，相当于重新创建了一个String对象。每个用双引号括起来的字符串都是String类的一个实例。
在比较字符串内容是否相等时，需要使用`equals()`方法。如果使用`==`运算符检测两个字符串，只能判断内存地址值是否一致（同一对象）。
字符串只有字面量是共享的，虚拟机不会始终将字符串共享，`+`或`substring`等操作得到的字符串并不共享。所以如果使用`==`运算符比较字符串，将造成迷惑的Bug。

## charAt()
`char charAt(int index)`

返回指定索引处的`char`值。

### 查找字符
```java
public void charAtTest() {
    String str = "LingYYY";
    System.out.println("str = " + str);

    // 查找字符
    char target = 'Y';
    List<Integer> index = new ArrayList<>();
    for (int i = 0; i < str.length(); i++) {
        if (str.charAt(i) == 'Y') {
            index.add(i);
        }
    }
    System.out.println("index = " + index);
}

// console
// str = LingYYY
// index = [4, 5, 6]
```

## substring
`String substring(int beginIndex)`
截取字符串。从下标为beginIndex开始（包括）到字符串最后。

`String substring(int beginIndex, int endIndex)`
截取字符串。从下标为beginIndex开始（包括）到endIndex结束（不包括）

### 使用示例

```java
public void subStringtTest() {
    String str = "LingYYY";
    System.out.println("str = " + str);

    System.out.println("str.substring(2) = " + str.substring(2));
    System.out.println("str.substring(1,4) = " + str.substring(1, 4));
}

// console
// str = LingYYY
// str.substring(2) = ngYYY
// str.substring(1,4) = ing
```

## concat
`String concat(String str)`
字符串拼接函数，连接两个字符串。

### 使用示例

```java
public void concatTest() {
    String str = "LingYYY";
    System.out.println("str = " + str);

    str = str.concat("uan");
    System.out.println("str = " + str.concat("!!!"));
}
// console
// str = LingYYY
// str = LingYYYuan!!!
```

## split
`String[] split(String regex)`
使用正则表达式分割字符串。

`String[] split(String regex, int limit)`
使用正则表达式分割字符串，并指定返回份数。

### 使用示例

```java
public void splitTest() {
    String keyword = "维达 纸巾      130抽";
    String[] keywords = keyword.split("\\s+");
    System.out.println("keywords = " + Arrays.toString(keywords));

    String[] keywords2 = keyword.split("\\s+", 2);
    System.out.println("keywords2 = " + Arrays.toString(keywords2));
}
// console
// keywords = [维达, 纸巾, 130抽]
// keywords2 = [维达, 纸巾      130抽]
```

## join
`String join(CharSequence delimiter, CharSequence... elements)`
使用delimitter合并拼接elements。delimitter是界定分割符，elements是多个待拼接参数。

`String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)`
使用delimitter合并拼接elements。delimitter是界定分割符，elements要求是`CharSequence`的实现类，包括但不仅限于`List`，`Array`

### 使用示例
```java
public void joinTest() {
    String sizeStr = String.join(" | ", "S", "M", "L", "XL");
    System.out.println("sizeStr = " + sizeStr);
    
    String keyword = "维达 纸巾      130抽";
    String[] keywords = keyword.split("\\s+");
    String keywordJoinStr = String.join(" | ", keywords);
    System.out.println("keywordJoinStr = " + keywordJoinStr);    
}
```
## repeat()
`String repeat(int count)`
复制当前字符串`count`次。JDK11新方法。

### 使用示例
```java
public void repeatTest() {
    String str = "Hello World!\n";
    System.out.println(str.repeat(3));
}
```
## isEmpty
`boolean isEmpty(String str)`
判断字符串是否为空字符串，即长度为0的字符串。

## StringBuilder & StringBuffer
`StringBuilder`是带缓冲区的可变字符串，字符串拼接时，不会生成新的对象。遇到大量字符串处理一定要使用。其前身是`StringBuffer`。
`StringBuffer`线程安全但效率稍低。如果所有字符串编辑操作都在单线程中执行，应使用`StringBuilder`。

有时需要由较短的字符串构建字符串，如果采用字符串拼接的方法效率较低，由于字符串不可变，每次拼接都会产生一个`String`对象，耗时且浪费空间。

`StringBuilder`和`StringBuffer`共享 API。

| toString() | 转为String类型 |
| --- | --- |
| append() | 从尾部添加内容 |

### 使用示例

```java
public void stringBuffer() {
	String str = new String("abc");//创建两个对象:‘abc’、‘String’
    String str1 = "abc";//创建一个对象:‘abc’
    str1 = str+str1;//创建一个新对象:‘abcabc’

    StringBuffer sb = new StringBuffer(str1);
    sb.append(str);
    System.out.println(sb);

	//删除
    sb.delete(2,3);
    System.out.println(sb);

	//插入
        sb.insert(2,"中国");
        System.out.println(sb);

	//替换
    sb.replace(3,5,"bababa");
    System.out.println(sb);

	//效率测试
	//Date对象获取当前电脑时间
    System.out.println("String拼接开始:"+new Date());
    String strs="";
    for (int i=0;i<50000;i++){
        strs+=i;
    }
	System.out.println("String拼接结束:"+new Date());

    System.out.println("StringBuffer拼接开始:"+new Date());
    System.out.println("StringBuffer");
    StringBuffer sb1 = new StringBuffer("");
    for (int i=0;i<50000;i++){
        sb1.append(i);
    }
    System.out.println("StringBuffer拼接结束:"+new Date());
}
```


## StringAPI
| charAt() | 返回当前索引处的字符 |
| --- | --- |
| compareTo() | 按字典顺序比较字符串大小 |
| equals() | 比较内容 |
| equalsIgnoreCase() | 忽略大小写比较 |
| indexOf() | 返回字符串第一次出现的位置 |
| replace() | 替换 |
| replaceAll() | 替换所有 |
| split() | 根据正则匹配拆分字符串 |
| substring() | 根据指定范围截取 |
| toCharArray() | 转换为字符数组 |
| toUpperCase() | 转大写 |
| toLowerCase() | 转小写 |
| trim() | 去前后空格 |
| length() | 返回字符串长度 |
| isEmpty | 判断字符串是否为空 |

## 统计每个字符出现次数
```java
 /**
 * 计字符串中每个字符出现次数
 * */
public static void solution() {
    // 创建一个随机字符串
    String str = createRandomStr(10);
    System.out.println(str);

    // 遍历字符串
    for (int i = 0; i < str.length(); i++) {
        // 要统计的字符
        char c = str.charAt(i);
        // 出现的次数
        int num = 0;

        // 是否统计过
        boolean flag = true;
        //判断在 i 之前有没有出现过该字符 , 如出现说明已统计过, 结束这个字符的循环
        for (int y = 0; y < i; y++) {
            if (c == str.charAt(y)) {
                flag = false;
                break;
            }
        }
        // true说明没有出现过, 需要统计
        if (flag) {
            //将字符和字符串中每一个字符比较
            for (int n = 0; n < str.length(); n++) {
                if (c == str.charAt(n)) {
                    num++;
                }
            }
            System.out.println(c + "出现了" + num + "次");
        }
    }
}
```

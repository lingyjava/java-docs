# 3·Java数组

- [3·Java数组](#3java数组)
  - [数组是什么](#数组是什么)
  - [存储结构](#存储结构)
  - [索引](#索引)
  - [API](#api)
  - [多维数组](#多维数组)
  - [Arrays](#arrays)
  - [数组排序](#数组排序)
    - [随机数组生成](#随机数组生成)
    - [冒泡排序](#冒泡排序)
    - [选择排序](#选择排序)
  - [数组扩容](#数组扩容)
  - [稀疏数组](#稀疏数组)


## 数组是什么
数组是一种数据容器，存储相同类型数据的有序集合。
数组是一个对象，属于引用类型，每个元素相当于对象中的成员变量。

## 存储结构
数组在定义时，申请的内存地址是连续的，在访问任意下标时都可以很高的效率，**适合用于随机访问**，但由于长度不可变，**不适合添加元素。**

![图2-数组内存结构](/docs/images/图2-数组内存结构.jpeg)

因此数组一经分配空间，其中的每个元素也被按照实例变量同样的方式被隐式初始化。一旦定义长度不可变，只能通过创建新数组转移元素进行数组的扩容。

## 索引
数组元素通过索引访问，索引从0开始。所以下标的合法区间是`[0，length-1]`，如果越界则抛出数组下标越界异常`ArraylndexOutOfBoundsException`。
![图3-数组下标结构](/docs/images/图3-数组下标结构.jpeg)

## API
数组对象提供的方法和常量。

| length | 数组长度 |
| --- | --- |

## 多维数组
多维数组是指数组中的元素也是一个数组。
使用场景有**动态规划。**
```java
public static void chessboard() {
    // 创建一个二维数组 11 * 11  表示棋盘
    // 0：没有棋子  1：黑棋  2：白棋
    int[][] arr = new int[11][11];
    arr[1][2] = 1;
    arr[2][3] = 2;
    
    // 输出棋盘
    System.out.println("输出原始的数组");
    for (int[] ints : arr) {
        for (int anInt : ints) {
            System.out.print(anInt + "\t");
        }
        System.out.println();
    }
}
```

## Arrays
数组对象本身没有什么方法，提供了工具类Arrays使用，对数据对象进行一些基本的操作。
`java.util.Arrays`是数组工具类，其方法都是static修饰的静态方法。

| sort() | 将数组升序排序 |
| --- | --- |
|  |  |

## 数组排序
数组排序算法是入门级算法。

### 随机数组生成
使用`Math.Random()`随机函数生成随机的数组。
```java
/**
 * 创建整型数组并返回
 * @param length 随机数组的长度
 * @return 整型数组
 * */
public int[] createIntArr(int length) {
    int[] array = new int[length];
    for(int i = 0; i < length; i++) {
        array[i] = (int) (Math.random() * 100);
    }
    return array;
}
```

### 冒泡排序
经典算法，比较数组相邻的数组，每轮循环至少找到一个元素到正确的位置。
```java
/**
 * 冒泡排序
 * */
public void bubbleSort(){
    int[] arr = createIntArr(10);
    
    for (int i = 0; i < arr.length - 1; i++){
        for (int j = 0; j < arr.length - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
    for (int ele : arr) {
        System.out.print(ele + "\t");
    }
}
```

### 选择排序
```java
/**
 * 选择排序
 *  */
public void selectSort(){
    int[] arr = createIntArr(10);

    for (int i = 0; i < arr.length - 1; i++){
        for (int j = i + 1; j < arr.length; j++){
            if (arr[i] > arr[j]){
                int temp = arr[j];
                arr[j] = arr[i];
                arr[i] = temp;
            }
        }
    }
    for (int ele : arr) {
        System.out.print(ele + "\t");
    }
}
```

## 数组扩容
数组的长度在创建时固定，如果还需要加入元素则需要进行数组扩容，在基本数组使用的数据结构ArrayList中就实现了自动数组扩容，原理如下：

```java
/**
 * 数组扩容
 * */
public void arrayExpansion(){
    String[] products = new String[5];

    Scanner sc = new Scanner(System.in);
    int number = 1;
    while(true){
        System.out.println("请输入第【" + number + "】个订单商品(输入end结束):");
        String product = sc.nextLine();
        if ("end".equals(product)){
            break;
        } else {
            if (number <= products.length){
                products[number - 1] = product;
            } else {
                // 数组长度不够需要扩容
                String[] newArr = new String[products.length + (products.length / 2)];

                // 数组复制
                System.arraycopy(products, 0, newArr, 0, products.length);
                
                newArr[number - 1] = product;
                products = newArr;
            }
            number++;
        }
    }
    System.out.println("本次订单商品:【" + Arrays.toString(products) + "】");
}
```
## 稀疏数组
当一个数组中大部分元素为相同的值时，使用稀疏数组保存数组可以节省空间。

稀疏数组的处理方法是：
1. 记录数组一共有几行几列，有多少个不同值。
2. 把具有不同值的元素和行列及值记录在一个小规模的数组中，从而缩小程序的规模。

```java
public void sparseArray() {
    // 创建一个二维数组11*11 , 模拟棋盘
    // 0：没有棋子  1：黑棋  2：白棋
    int[][] arr = new int[11][11];
    arr[1][2] = 1;
    arr[2][3] = 2;

    //输出原始的数组
    System.out.println("输出原始的数组");
    for (int[] ints : arr) {
        for (int anInt : ints) {
            System.out.print(anInt + "\t");
        }
        System.out.println();
    }

    /*     转换为稀疏数组保存    */

    // 有效值的个数
    int effectiveValue = 0;
    for (int i = 0; i < 11; i++) {
        for (int j = 0; j < 11; j++) {
            if (arr[i][j] != 0) {
                effectiveValue++;
            }
        }
    }

    // 创建一个稀疏数组的二维数组
    // 第一行存放二维数组的长宽和有效值个数, 之后一个有效值一行 ,
    int[][] arr2 = new int[effectiveValue + 1][3];
    arr2[0][0] = 11;
    arr2[0][1] = 11;
    arr2[0][2] = effectiveValue;

    //遍历二维数组，将非零的值，存放在稀疏数组中
    int count = 0;
    for (int i = 0; i < arr.length; i++) {
        for (int j = 0; j < arr.length; j++) {
            if (arr[i][j]!=0){
                count++;
                // 每行记录三个值（x坐标,y坐标,实际值）
                arr2[count][0] = i;
                arr2[count][1] = j;
                arr2[count][2] = arr[i][j];
            }
        }
    }
    System.out.println("稀疏数组：");
    for (int i = 1; i < arr2.length; i++) {
        System.out.println(arr2[i][0]+"\t"+
                arr2[i][1]+"\t"+
                arr2[i][2]+"\t");
    }

    //还原稀疏数组
    int[][] arr3 = new int[arr2[0][0]][arr2[0][1]];
    // 给其中的元素还原它的值
    for (int i = 1; i < arr2.length; i++) {
        arr3[arr2[i][0]][arr2[i][1]] = arr2[i][2];
    }
    System.out.println("还原的数组");
    for (int[] ints : arr3) {
        for (int anInt : ints) {
            System.out.print(anInt + "\t");
        }
        System.out.println();
    }
}

```

# 1·IO流常用类

- [1·IO流常用类](#1io流常用类)
  - [IO流是什么](#io流是什么)
  - [IO流体系](#io流体系)
  - [File](#file)
  - [字节流相关](#字节流相关)
  - [实现逐行输出文本文件的内容](#实现逐行输出文本文件的内容)

## IO流是什么
IO流就是input和output数据流向，向文件中写入数据和读取数据。
包括：
1. 输入输出流。
2. 字节字符流
3. 包装流。

Java 的 I/O 大概可以分成以下几类:
- 磁盘操作: File
- 字节操作: InputStream 和 OutputStream
- 字符操作: Reader 和 Writer
- 对象操作: Serializable
- 网络操作: Socket

## IO流体系
![图4-io知识体系](/docs/images/图4-io知识体系.jpg)

## File
File 类可以用于表示文件和目录的信息，但是它不表示文件的内容。
```java
public static void listAllFiles(File dir) {
    if (dir == null || !dir.exists()) {
        return;
    }
    if (dir.isFile()) {
        System.out.println(dir.getName());
        return;
    }
    for (File file : dir.listFiles()) {
        listAllFiles(file);
    }
}
```

## 字节流相关
```java
public static void copyFile(String src, String dist) throws IOException {

    FileInputStream in = new FileInputStream(src);
    FileOutputStream out = new FileOutputStream(dist);
    byte[] buffer = new byte[20 * 1024];

    // read() 最多读取 buffer.length 个字节
    // 返回的是实际读取的个数
    // 返回 -1 的时候表示读到 eof，即文件尾
    while (in.read(buffer, 0, buffer.length) != -1) {
        out.write(buffer);
    }

    in.close();
    out.close();
}
```

## 实现逐行输出文本文件的内容
```java
public static void readFileContent(String filePath) throws IOException {

    FileReader fileReader = new FileReader(filePath);
    BufferedReader bufferedReader = new BufferedReader(fileReader);

    String line;
    while ((line = bufferedReader.readLine()) != null) {
        System.out.println(line);
    }

    // 装饰者模式使得 BufferedReader 组合了一个 Reader 对象
    // 在调用 BufferedReader 的 close() 方法时会去调用 Reader 的 close() 方法
    // 因此只要一个 close() 调用即可
    bufferedReader.close();
}
```

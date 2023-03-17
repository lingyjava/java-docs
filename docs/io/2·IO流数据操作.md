# 2·IO流数据操作

- [2·IO流数据操作](#2io流数据操作)
  - [数据传输](#数据传输)
  - [字符流对象](#字符流对象)
  - [字节流对象](#字节流对象)
  - [字节流和字符流的区别](#字节流和字符流的区别)
    - [字节转字符](#字节转字符)
  - [数据操作](#数据操作)
  - [Scanner对象](#scanner对象)
  - [使用示例](#使用示例)

## 数据传输
从数据传输方式可以将 IO 类分为：字节流，字符流。
字节是个计算机看的，字符是给人看的。

## 字符流对象
![图5-字符流对象](/docs/images/图5-字符流对象.png)

## 字节流对象
![图6-字节流对象](/docs/images/图6-字节流对象.png)

## 字节流和字符流的区别
- 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码是 3 个字节，中文编码是 2 个字节。)
- 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，具有可读性)。

### 字节转字符
编码就是把字符转换为字节，而解码是把字节重新组合成字符。
如果编码和解码过程使用不同的编码方式那么就出现了乱码。

编码规则：
- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。

Java 使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。

## 数据操作
![图7-数据操作对象](../images/图7-数据操作对象.png)

## Scanner对象
Scanner对象可以获取用户的输入。

```java
Scanner sc = new Scanner(System.in);
sc.next();//next()遇到带有空格的字符串会自动将其去掉,要读取到有效字符后才可以结束输入
sc.nextLine();//可以获得空白,以Enter为结束符
sc.nextInt();//输入一个整型
if(sc.hasNext){//判断是否有输入
    String str = sc.next();
    System.out.println("输入的内容为："+str);
}
//IO流的类如果不关闭会一直占用资源,用完就关掉
sc.close();
```

## 使用示例
```java
import com.example.bean.Person;
import java.io.*;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class TestIO {
    public void IO() {
//        复制文件
        File oldFile = new File("D:/awork/oldFile.txt");
        File newFile = new File("D:/newFile1.txt");

//        创建输入输出流
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            Date d1 = new Date();
            fis = new FileInputStream(oldFile);
            fos = new FileOutputStream(newFile);

//        将输入流里面的数据导入到输出流中
//        字节输入输出流
            byte[] b = new byte[1024*1024];
            int index=-1;
            while((index=fis.read(b))!=-1){
                fos.write(b,0,index);
                System.out.println();
            }
            Date d2 = new Date();
            fos.flush();
            System.out.println("复制完成！耗时："+(d2.getTime()-d1.getTime())+"毫秒");

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try{
                if (fis!=null){
                    fis.close();
                }
                if (fos!=null){
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

//      字符流输入输出流
        File oldFile1 = new File("D:/awork/oldFile.txt");
        File newFile1 = new File("D:/newFile1.txt");
//        创建字符输入输出流
        FileReader fr=null;
        FileWriter fw=null;

        try {
            fr=new FileReader(oldFile1);
            fw=new FileWriter(newFile1);
//            通过输入流获取文件数据，通过输出流把数据写到文件中
            char[] c = new char[1024];
            int len = -1;
            while((len=fr.read(c))!=-1){
                fw.write(c,0,len);
            }
//        刷新
            fw.flush();
            System.out.println("复制完成");
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if (fr!=null) {
                    fr.close();
                }
                if (fw!=null){
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        /*
        * 包装流(过滤流):在基本流的基础上提供了一些特殊的概念
        *BufferedReader:一行一行读的字符包装类
        * PrintWrite:一行一行写的字符包装类
        * */

        File oldFile2 = new File("D:/awork/oldFile.txt");
        File newFile2 = new File("D:/awork/newFile.txt");

        FileReader fr1=null;
        FileWriter fw1=null;
        BufferedReader br = null;
        PrintWriter pw = null;

        try {
            fr1 = new FileReader(oldFile2);
            fw1 = new FileWriter(newFile2,true);//true参数要求复制的数据追加到文件后
            br = new BufferedReader(fr1);
            pw = new PrintWriter(fw1,true);//true参数要求复制的数据追加到文件后
            String str = null;
            while ((str=br.readLine())!=null){
                pw.write(str);
            }
            pw.flush();
            System.out.println("复制完成");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
//        只需要关闭包装流，基本流会自动关掉
            try{
                if (pw!=null){
                    pw.close();
                 }
                if (br!=null){
                    br.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        /*
        * 对象输入输出流
        * ObjectInputStream
        * ObjectOutputStream
        * */

        Person p1 = new Person("张三",18,80);
        Person p2 = new Person("李四",18,80);
        Person p3 = new Person("王五",18,80);

        List<Person> ps = new ArrayList<Person>();
        ps.add(p1);
        ps.add(p2);
        ps.add(p3);

        File file = new File("D:/aword/oldFile.txt");

        FileOutputStream fos1 = null;
        ObjectOutputStream oos = null;

        try {
            fos = new FileOutputStream(file);
            oos = new ObjectOutputStream(fos);
            oos.writeObject(ps);
            oos.flush();
            System.out.println("保存完成");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try{
                if (oos!=null){
                    oos.close();
                }
            }catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

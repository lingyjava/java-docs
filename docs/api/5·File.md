# 5·File

- [5·File](#5file)
  - [File对象是什么](#file对象是什么)
  - [API](#api)
  - [使用示例](#使用示例)

## File对象是什么
`File`对象用于表示硬盘上的一个文件夹或文件。提供操作文件或文件夹的函数。

## API
| createNewFile() | 创建文件 |
| --- | --- |
| canRead() | 判断是否只读 |
| canWrite() | 判断是否只写 |
| getAbsolutePath() | 判断是否存在并返回地址 |
| getName() | 获得文件或目录名 |
| getParent() | 获得父文件或目录名 |
| getPath() | 获取文件或文件夹路径 |
| isDirectory() | 判断是否是目录 |
| isFile() | 判断是否是文件 |
| lastModified() | 获取最后修改时间 |
| length() | 获取文件或文件夹长度，空文件夹长度为0，win系统文件夹不占空间 |
| list() | 获取目录下所有子目录和子文件名 |
| mkdir() | 创建一层目录，返回成功状态 |
| mkdirs() | 创建多层目录 |
| delete() | 删除空文件夹 |

## 使用示例

```java
import java.io.File;
import java.io.IOException;

public class TestFile {
    /*
    * File:用于表示硬盘上的一个文件夹或文件
    * */
    //通过构造方法创建文件:表示硬盘上的目录
    File file = new File("D:/awork/File/oldFile.txt");
    public void test1() {
        //新建文件
        try {
            file.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public void test3(){
        //获取该目录下所有子目录子文件的名字
        String[] names = file.list();
        for (String str:names) {
            System.out.println(str);
        }
    }
    public void test4() {
        //创建一层目录目录
        File newFileDir = new File("D:/awork/fileDir");
        boolean mkdir = newFileDir.mkdir();
        System.out.println(newFileDir.mkdir());//true
    }
    public void test5(){
        //创建多层目录
        File fileDirs = new File("D:/awork/File/aa/bb/cc");
        fileDirs.mkdirs();
    }
    public void test6(){
        //删除空文件夹
        File fileDleDir = new File("D:/awork/File/aa/");
        fileDleDir.delete();
    }
}
```

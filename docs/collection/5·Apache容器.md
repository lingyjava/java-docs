# 5·Apache容器

- [5·Apache容器](#5apache容器)
  - [为什么使用Apache容器](#为什么使用apache容器)
  - [Maven依赖](#maven依赖)
  - [容器](#容器)
  - [使用示例](#使用示例)

## 为什么使用Apache容器
可以用于返回多个对象，节省实体类的使用。

## Maven依赖
```xml
<!-- apache工具 -->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.11</version>
</dependency>
```

## 容器
`Pair<Objecr,Object>`：可保存两个对象。
`Triple<Object,Object>`：可保存三个对象。

## 使用示例
```java
package com.ly.example;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.commons.lang3.tuple.ImmutablePair;
import org.apache.commons.lang3.tuple.ImmutableTriple;
import org.apache.commons.lang3.tuple.Pair;
import org.springframework.stereotype.Service;

/**
 * @author ly
 * @Date: 2022/5/18 22:42
 */
@Service(value = "apacheContainer")
public class ApacheContainer {

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    class Entity{
        private Integer id;
        private String message;
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    class Pojo{
        private Integer status;
        private Entity data;
    }
    
    public ImmutablePair<Entity,Pojo> getPair(){
        return ImmutablePair.of(new Entity(),new Pojo());
    }

    public ImmutableTriple<Entity,Pojo,String> getTriple(){
        Entity entity = new Entity(200,"success");
        Pojo pojo = new Pojo(200,entity);
        return ImmutableTriple.of(new Entity(),new Pojo(),"666");
    }

    public Object method(ImmutableTriple<Entity,Pojo,String> obj){
        ImmutableTriple<Entity,Pojo,String> immutableTriple = getTriple();
        // 获取左边的对象
        Entity entity = immutableTriple.getLeft();
        // 获取中间的对象
        Pojo pojo = immutableTriple.getMiddle();
        // 获取右边的对象
        String msg = immutableTriple.getRight();
        return immutableTriple;
    }
}
```

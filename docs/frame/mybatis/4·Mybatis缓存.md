# Mybatis缓存

- [Mybatis缓存](#mybatis缓存)
  - [为什么需要缓存](#为什么需要缓存)
  - [二级缓存](#二级缓存)

## 为什么需要缓存
数据在某一段时间需要进行大量重复查询时，可以将查询结果存到缓存当中。

Mybatis缓存模型：
![图14-Mybatis缓存模型](/docs/images/图14-Mybatis缓存模型.jpeg)

## 二级缓存
Mybatis中提供了两种缓存方式。
一级缓存，默认开启。
Mybatis给每个SqlSession对象提供了一个一级缓存空间，同一个SqlSession执行同一条SQL语句，第二条SQL语句会从缓存中获取，第一条语句查询出结果后将结果放到一级缓存中。
二级缓存，手动开启。
开启后Mybatis会给每个命名空间提供一个二级缓存空间，不同的SqlSession操作同一命名空间下的同一条SQL语句时共享二级缓存。第一个SqlSession必须提交事务后，所查询的数据才会放到二级缓存中。

1. 在Mybatis主配置文件中开启二级缓存。
```xml
<configuration>
	<settings>
    <!--手动开启二级缓存-->
    <setting name="cacheEnabled" value="true"></setting>
  </settings>
</configuration>


```

2. 在映射文件中确认开启二级缓存。
```xml
<mapper namespace="com.java.pojo.emp">
  <!--在需要进行二级缓存的命名空间中开启二级缓存-->
  <cache></cache>
</mapper>
```

3. 需要缓存的对象必须实现可序列化接口。
```java
public class Emp implements Serializable{}
```

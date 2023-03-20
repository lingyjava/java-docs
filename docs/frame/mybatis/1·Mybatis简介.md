# 1·Mybatis简介

- [1·Mybatis简介](#1mybatis简介)
  - [Mybatis是什么](#mybatis是什么)
  - [为什么使用Mybatis](#为什么使用mybatis)
  - [依赖](#依赖)
  - [主配置文件](#主配置文件)
  - [数据库连接信息](#数据库连接信息)
  - [原生使用](#原生使用)

## Mybatis是什么
Mybatis属于一种ORM（Object Relational Mapping）框架，持久层框架，支持自定义 SQL、存储过程以及高级映射。免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

- [mybatis中文文档](https://mybatis.org/mybatis-3/zh/index.html)

## 为什么使用Mybatis
JDBC中存在大量重复的操作，结果集需要封装处理，SQL语句嵌入在Java代码中，使用直连数据库（效率低）。
Mybatis底层是基于JDBC的封装，用来替代JDBC，相比于JDBC的优点：

1. Mybatis使用数据库连接池，在List<Connection>集合中存放数据库连接，使用时取出。
2. SQL语句在映射文件中，与Java代码分离。
3. 自动处理结果集。

![图13-Mybatis工作流程](/docs/images/图13-Mybatis工作流程.jpeg)

## 依赖
导入依赖。
```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

## 主配置文件
配置主配置文件。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<!--导入外部配置文件,jdbc的配置文件-->
	<properties resource="jdbc.properties"/>

	<!--允许mybatis插入null,mybatis会自动将null转换为空字符串-->
	<settings>
		<setting name="jdbcTypeForNull" value="NULL"/>
	</settings>

	<!--给实体类起别名,方便在映射文件中使用,别名只能在CRUD标签中使用-->
	<typeAliases>
		<typeAlias type="com.java.pojo.User" alias="user"/>
		<typeAlias type="com.java.pojo.Dept" alias="dept"/>
		<typeAlias type="com.java.pojo.Emp" alias="emp"/>
		<typeAlias type="com.java.util.FenYeUtil" alias="fy"/>
		<typeAlias type="com.java.pojo.Query" alias="query"/>
	</typeAliases>

	<!--配置数据库连接池生产环境-->
	<environments default="development">
		<environment id="development">
			<!-- 使用JDBC的事务管理方式 -->
			<transactionManager type="JDBC"></transactionManager>
			<!-- 使用mybatis自己的连接池 -->
			<dataSource type="POOLED">
	 			<property name="driver" value="${jdbc.driver}"/>
					<property name="url" value="${jdbc.url}"/>
					<property name="username" value="${jdbc.username}"/>
					<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
	</environments>

	<!--加载映射文件-->
	<mappers>
		<mapper resource="com/java/dao/UserMapper.xml"/>
		<mapper resource="com/java/dao/DeptMapper.xml"/>
		<mapper resource="com/java/dao/EmpMapper.xml"/>
	</mappers>
	
</configuration>
```

## 数据库连接信息
配置数据库连接信息。
```properties
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc.username=username
jdbc.password=password
```

- [JDBC: 连接不同的数据库的Driver和URL的写法](https://blog.csdn.net/isomebody/article/details/73012417)

## 原生使用
获取数据库连接工具类。
```java
//单例工具类：用于产生数据库连接
public class MyBatisUtil {
    //SqlSessionFactory对象,类似于DriverManager用来产生连接
    private SqlSessionFactory sb;
    private static MyBatisUtil mu = new MyBatisUtil();
    
    private MyBatisUtil(){
        InputStream is = this.getClass().getClassLoader().getResourceAsStream("mybatis.xml");
        sb = new SqlSessionFactoryBuilder().build(is);
    }
    public static MyBatisUtil init(){
        return mu;
    }
    public SqlSession getSqlSession(){
        return sb.openSession();
    }
}
```
获取连接并执行SQL.
```java
public class EmpServiceImpl implements EmpService {
    @Override
    public int deleteEmpByEmpNo(String[] empNos) {
        //SqlSession对象，类似于Connection产生数据库连接
        //获得一个数据库连接
        SqlSession ss = MybatisUtil.init().getSqlSession();
        //使用getMapper()方法为EmpDao生成实现类
        EmpDao ed = ss.getMapper(EmpDao.class);
        if (empNos != null) {
            for (String empNo : empNos) {
                ed.deleteEmpByEmpNo(new Integer(empNo));
            }
        }
        //mybatis默认为手动提交
        ss.commit();
        //关闭数据库连接
        ss.close();
        return 1;
    }

    @Override
    public List<Emp> queryAllManager() {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        List<Emp> emps = ed.queryAllManager();
        ss.close();
        return emps;
    }

    @Override
    public int insertEmp(Emp emp) {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        int flag = ed.insertEmp(emp);
        ss.commit();
        ss.close();
        return flag;
    }

    @Override
    public Emp queryEmpByEmpNo(int empNo) {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        Emp emp = ed.queryEmpByEmpNo(empNo);
        ss.close();
        return emp;
    }

    @Override
    public int updateEmpByEmpNo(Emp emp) {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        int flag = ed.updateEmpByEmpNo(emp);
        ss.commit();
        ss.close();
        return flag;
    }

    @Override
    public List<Emp> queryByFy(FenYeUtil fy, String pg, Query query) {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        //查询符合条件的记录总数
        fy.setRowsCount(ed.queryEmpCountByQuery(query));
        int pgnum = 1;
        if (pg != null && !"".equals(pg)){
            pgnum = new Integer(pg);
        }
        if (pgnum < 1) {
            pgnum = 1;
        }
        if (pgnum > fy.getRowsCount()){
            pgnum = fy.getPagesCount();
        }
        fy.setPg(pgnum);
        fy.setQuery(query);
        List<Emp> emps = ed.queryByFy(fy);
        ss.close();
        return emps;
    }

    @Override
    public int queryEmpCountByQuery(Query query) {
        SqlSession ss = MybatisUtil.init().getSqlSession();
        EmpDao ed = ss.getMapper(EmpDao.class);
        int num = ed.queryEmpCountByQuery(query);
        ss.close();
        return num;
    }

}
```
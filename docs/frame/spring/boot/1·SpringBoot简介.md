# 1·SpringBoot简介

- [1·SpringBoot简介](#1springboot简介)
  - [SpringBoot-是什么？](#springboot-是什么)
  - [配置文件](#配置文件)
  - [依赖](#依赖)
  - [程序入口常用注解](#程序入口常用注解)

## SpringBoot-是什么？
Spring Boot框架，设计目的是简化Spring应用的创建、运行、调试、部署等。使用Spring Boot可以做到专注于Spring应用的开发，而无需过多关注XML的配置。
使用“习惯优于配置”的理念。它提供了一堆依赖打包，并已经按照使用习惯解决了依赖问题。使用Spring Boot可以不用或者只需要很少的Spring配置就可以让企业项目快速运行起来。帮助开发者统筹管理应用的配置，提供基于实际开发中常见配置的默认处理，简化应用的开发，简化应用的运维。
Spring Boot的精简理念又使其成为Java微服务（分布式）开发的不二之选，也可以说，Spring Boot其实就是为了微服务而生的Java web框架。
极大简化的Maven配置：Spring提供推荐的基础 POM 文件来简化Maven 配置。
自动配置Spring：Spring Boot会根据项目依赖来自动配置Spring 框架，极大地减少项目要使用的配置。
提供生产就绪型功能：提供可以直接在生产环境中使用的功能，如性能指标、应用信息和应用健康检查。
无代码生成和xml配置：Spring Boot不生成代码。完全不需要任何xml配置即可实现Spring的所有配置。

## 配置文件
SpringBoot中简化了框架的配置文件，所有的配置写在application.properties或application.yml文件中。

## 依赖
- `Spring Boot DevTools`：项目热部署
- `Spring Web`：web模块
- `Mybatis Framework`：Mybatis框架
- `Spring Data Redis(Access+Driver)`：redis数据库
- `Apache Freemarker`：freemarker模板
- `Eureka Discovery Client`：eureka客户端
- `Oracle Driver`：Oracle驱动

```xml
<!--springboot解析Jsp页面依赖-->
<dependency>
		  <groupId>org.apache.tomcat.embed</groupId>
		  <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
<dependency>
		  <groupId>javax.servlet</groupId>
		  <artifactId>jstl</artifactId>
</dependency>

<!--springboot热部署依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
<!--开启springboot热部署-->
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
      	<fork>true</fork><!--该配置必须-->
      </configuration>
    </plugin>
  </plugins>
</build>

<!--Oracle数据库依赖,此依赖为本地仓库依赖-->
<dependency>
  <groupId>com.oracle</groupId>
  <artifactId>ojdbc6</artifactId>
  <version>11.2</version>
</dependency>

<!--Freemarker模板依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>

<!--文件上传依赖-->
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.2.2</version>
</dependency>
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.4</version>
</dependency>

<!--Redis数据库依赖,导入依赖后必须配置连接属性,否则报错-->
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--session共享依赖-->
<dependency>  
  <groupId>org.springframework.session</groupId>  
  <artifactId>spring-session-data-redis</artifactId>  
</dependency> 

<!--定时任务Quartz依赖-->
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
</dependency>
```

## 程序入口常用注解
```java
@SpringBootApplication//表示程序入口
@ComponentScan("com.ly")//开启注解扫描
@MapperScan("com.ly.dao")//声明需要代理生成实现类的接口,dao层所在目录
@ServletComponentScan("com.ly.filter")//声明servlet相关的api的目录
@EnableTransactionManagement//打开事务管理注解开发功能
@EnableRedisHttpSession//开启session共享
@EnableScheduling//开启任务调度
```

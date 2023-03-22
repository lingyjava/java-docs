# 3·Maven

- [3·Maven](#3maven)
  - [Maven是什么](#maven是什么)
  - [setting.xml详解](#settingxml详解)
  - [配置镜像地址](#配置镜像地址)
  - [pom文件详解](#pom文件详解)
  - [本地仓库依赖加载失败](#本地仓库依赖加载失败)
  - [依赖地址](#依赖地址)
    - [Servlet](#servlet)
    - [Jsp](#jsp)
    - [Mybatis](#mybatis)
    - [Spring](#spring)
    - [DB](#db)
    - [Log4j](#log4j)
    - [Lombok](#lombok)
    - [Hutool](#hutool)
    - [Junit](#junit)
  - [日志管理](#日志管理)
  - [安装与配置](#安装与配置)
  - [基础使用方式](#基础使用方式)
    - [统一版本管理](#统一版本管理)
    - [排除依赖](#排除依赖)
    - [jar包作用域](#jar包作用域)
    - [构建配置文件](#构建配置文件)
    - [项目分层](#项目分层)

## Maven是什么
自动化构建工具。按照规定的项目目录结构创建目录，就可以通过简单的命令完成构建工作。

jar包：一个Java项目打包成jar包。
war包：一个动态web项目打包成war包，放在tomcat的webapps中执行发布。

maven的功能：
1. 自动管理jar包：自动下载jar包，自动管理jar依赖问题。
2. 模块化开发。

## setting.xml详解
- [maven全局配置文件settings.xml详解](https://www.cnblogs.com/jingmoxukong/p/6050172.html)

## 配置镜像地址
- [Maven 镜像](https://developer.aliyun.com/mirror/maven)

## pom文件详解
- [maven之pom.xml配置文件详解](https://www.jianshu.com/p/0e3a1f9c9ce7)

## 本地仓库依赖加载失败
- [记 Maven 本地仓库埋坑之依赖包为何不能用](https://www.cnblogs.com/dasusu/p/11825786.html)

Q：内网开发时，无法下载外Q网依赖，于是在外网下载依赖后将maven仓库拷贝在内网电脑中，加载maven依赖失败。

1. 将Maven本地仓库里，拷过来的那个依赖包目录中，将 xxx.repositories 文件删掉，再重新构建项目即可。如果还是不能引用 这些 jar 的话 ， 删除 项目下的  .iml 文件，然后重启idea， 点击 maven 上面的刷新，就可以引用了。
2. 当maven的settings文件里面有mirror时，会在先加载mirror镜像里面的文件，不会加载本地仓库的文件。加载本地仓库的文件需要把mirror都注释掉。https://www.cnblogs.com/dasusu/p/11825786.html

## 依赖地址

### Servlet
```xml
<!--servlet-->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>sServlet-api</artifactId>
  <version>2.5</version>
  <scope>provided</scope>
</dependency>
```
### Jsp
```xml
<!--jsp-api-->
<dependency>
  <groupId>javax.servlet.jsp</groupId>
  <artifactId>jsp-api</artifactId>
  <version>2.1</version>
  <scope>provided</scope>
</dependency>

<!--JSTL-->
<dependency>
  <groupId>jstl</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```
### Mybatis
```xml
<!--mybatis-->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.2.7</version>
</dependency>
<dependency>
  <groupId>asm</groupId>
  <artifactId>asm</artifactId>
  <version>3.3.1</version>
  <scope>compile</scope>
</dependency>
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib-nodep</artifactId>
  <version>2.2</version>
  <scope>compile</scope>
</dependency>
```
### Spring
```xml
<!--spring-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-beans</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-expression</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>

<!-- mtabtis-spring整合依赖 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>1.2.2</version>
</dependency>

<!--springAOP-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aop</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>

<!--springWeb(springmvc)-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc-portlet</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-websocket</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>

<!--springmvc处理json-->
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.1.5</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.1.5</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <version>2.1.5</version>
</dependency>

<!-- spring事务管理依赖 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-orm</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
```
### DB
```xml
<!--Oracle数据库驱动依赖,此处是本地仓库依赖-->
<dependency>
  <groupId>com.oracle</groupId>
  <artifactId>ojdbc6</artifactId>
  <version>11.2</version>
</dependency>
<!--c3p0连接池-->
<dependency>
  <groupId>com.mchange</groupId>
  <artifactId>c3p0</artifactId>
  <version>0.9.2.1</version>
</dependency>

<!--redis-->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-pool2</artifactId>
  <version>2.4.2</version>
</dependency>
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-redis</artifactId>
  <version>1.6.0.RELEASE</version>
</dependency>

<!--Mysql-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.22</version>
</dependency>
```
### Log4j
```xml
<!--log4j-->
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.16</version>
</dependency>
```
### Lombok
```xml
<!--Lombok-->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.24</version>
</dependency>
```
### Hutool
```xml
<dependency>
  <groupId>cn.hutool</groupId>
  <artifactId>hutool-all</artifactId>
  <version>5.8.0.M4</version>
</dependency>
```
### Junit
```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13.2</version>
</dependency>
```
## 日志管理
```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.1.7</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.7</version>
</dependency>
```
## 安装与配置

1. maven安装与配置，安装目录不要有中文以及空格。
2. 配置环境变量：新建系统变量M2_HOME--->D:\apache-maven-3.3.9，在path里面添加%M2_HOME%\bin
3. 修改maven配置文件：settings.xml，修改本地仓库位置。

`<localRepository>D:\maven-local-repository</localRepository>`

4. 修改maven默认依赖jdk版本。
```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

5. idea项目设置与新项目设置本地maven仓库。
## 基础使用方式

1. maven使用坐标定位jar包，坐标由三部分组成。

组id：生产jar包的公司或者组织域名导致。
模块id：项目或者模块名称。
版本号：项目或者模块的版本号。

2. 将本地jar包导入maven仓库的命令。
```basic
Mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6   -Dversion=11.2 -Dpackaging=jar    -Dfile=D:\app\Administrator\product\11.2.0\dbhome_1\jdbc\lib\ojdbc6.jar

Mvn install:install-file -DgroupId=com.danga -DartifactId=memcached  -Dversion=2.6.3 -Dpackaging=jar    -Dfile=d:\Documents\Java51\Ebuy\lib\java_memcached-release_2.6.3.jar

Mvn install:install-file -DgroupId=mysql -DartifactId=mysql-connector-java  -Dversion=8.0.22 -Dpackaging=jar    -Dfile=d:\Documents\mysql-connector-java-8.0.22.jar
```

3. 在pom.xml中添加jar包坐标。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ly</groupId>
    <artifactId>testMaven</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <!--依赖文件:通过坐标导入需要的jar包-->
    <dependencies>
        <!--servlet-api-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
  </dependencies>
</project>
```
### 统一版本管理
在pom.xml文件中，每个依赖都有一个版本号，可以在配置文件中定义变量，在需要更换版本时，直接修改变量即可。
```xml
<properties>
	<servlet-api-version>2.5</servlet-api-version>
</properties>

<dependencies>
  <!--servlet-api-->
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>${servlet-api-version}</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
```
### 排除依赖
在依赖配置中排除指定的依赖。
```xml
<exclusions>
  <exclusion>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
  </exclusion>
</exclusions>
```
### jar包作用域
使用scpe标签定义依赖范围：

1. compile：主程序、测试程序及部署，可传递。
2. Test：测试程序(junit4)不参与部署，不可传递。
3. Provided：主程序测试程序，不参与部署(servlet-api，jsp-api)，不可传递。

依赖传递：
依赖路径最近原则，配置的依赖越近越优先，如果路径一致，先配置的越优先。
### 构建配置文件
maven默认将配置文件全部放在resources文件夹下，在src下的配置文件不会被构建。如果现在构建时加载src目录下的配置文件，需要做出配置。
```xml
<build>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.properties</include>
        <include>**/*.xml</include>
      </includes>
      <filtering>false</filtering>
    </resource>
    <resource>
      <directory>src/main/java</directory>
      <includes>
        <include>**/*.properties</include>
        <include>**/*.xml</include>
      </includes>
      <filtering>false</filtering>
    </resource>
  </resources>
</build>
```
### 项目分层
![图20-Maven分层架构](/docs/images/图20-Maven分层架构.jpeg)

子项目拥有父项目的所有配置文件(jar包)。

确定项目父子关系：

1. 父项目的pom配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ly</groupId>
    <artifactId>empParent</artifactId>

    <!--
    打包方式:
        pom:父项目
        jar:Java项目
        war:动态web项目
    -->
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <!--父项目通过modules标签引入子模块-->
    <modules>
        <module>pojo</module>
        <module>dao</module>
        <module>service</module>
        <module>controller</module>
    </modules>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
  
  <!--父项目的所有配置文件,公共的依赖-->

</project>
```

2. 子项目的pom配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!--指定父项目-->
    <parent>
        <artifactId>empParent</artifactId>
        <groupId>com.ly</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>pojo</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
  
  <!--默认拥有父项目的所有配置文件-->
  <!--配置子项目独有的配置文件...-->

</project>
```

3. 将web项目分为pojo、dao、service、controller层。

使dao导入pojo依赖，service导入dao依赖，controller导入service依赖。
实现依赖的传递。
```xml
<!--往dao项目加入一个pojo的jar包-->
<dependencies>
  <dependency>
    <groupId>com.ly</groupId>
    <artifactId>pojo</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
</dependencies>
```

4. 将mybatis主配置文件和jdbc.properties和login4j.properties放在dao模块。
5. 将spring相关配置文件和web模块放在controller模块。
6. 部署工件时，在WEB-INF目录下需要手动创建lib目录添加所有依赖项，在classes目录添加所有模块输出。

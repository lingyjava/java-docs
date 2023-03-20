# 6·SSM框架整合

- [6·SSM框架整合](#6ssm框架整合)
  - [SSM框架](#ssm框架)
  - [导入Jar包或依赖](#导入jar包或依赖)
  - [Mybatis主配置文件](#mybatis主配置文件)
  - [Spring主配置文件](#spring主配置文件)
  - [SpringMVC主配置文件](#springmvc主配置文件)
  - [数据库连接配置文件](#数据库连接配置文件)

## SSM框架
SSM：Spring，SpringMVC，Mybatis三个框架，整合SSM框架主要是将SpringMVC、Mybatis框架中涉及到的对象交给Spring框架管理。

[手把手教你如何玩转SSM框架整合(非Maven版本)](https://blog.csdn.net/Cs_hnu_scw/article/details/78157672)

## 导入Jar包或依赖
1. 数据库驱动。
2. Mybatis.
3. c3p0连接池。
4. SpringIOC.
5. SpringAOP.
6. Spring事务管理。
7. Mybatis和Spring整合。

## Mybatis主配置文件
配置Mybatis相关配置。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!--mybatis.xml	-->

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

	<!--加载映射文件-->
	<mappers>
		<mapper resource="com/java/dao/UserMapper.xml"/>
		<mapper resource="com/java/dao/DeptMapper.xml"/>
		<mapper resource="com/java/dao/EmpMapper.xml"/>
	</mappers>
	
</configuration>
```

## Spring主配置文件
配置Spirng主配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context-4.1.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.java"></context:component-scan>

    <!--导入jdbc资源文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置c3p0连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"></property>
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>

        <!--最小连接数-->
        <property name="minPoolSize" value="5"></property>
        <!--最大连接数-->
        <property name="maxPoolSize" value="20"></property>
        <!--最大空闲时间/s-->
        <property name="maxIdleTime" value="180"></property>
    </bean>

    <!--配置SqlSessionFactory-->
    <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--加载mybatis主配置文件-->
        <property name="configLocation" value="classpath:mybatis.xml"></property>
        <!--指定数据库连接池-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--让spring自动生成dao层接口实现类,并放到IOC容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--指定dao层接口位置-->
        <property name="basePackage" value="com.java.dao"></property>
        <!--设置SqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sessionFactory"></property>
    </bean>

</beans>
```

## SpringMVC主配置文件
配置MVC相关配置。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">


    <!--开启springMVC的注解开发功能-->
    <mvc:annotation-driven/>

    <!--配置jsp视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--拼接url-->
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
        <!--前缀,可以通过/WEB-INF/访问到WEB-INF文件夹下的文件-->
        <!--<property name="prefix" value="/"/>-->
    </bean>

    <!--排除静态资源-->
    <mvc:default-servlet-handler/>

</beans>
```

## 数据库连接配置文件
配置jdbc.properties配置文件。
```properties
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:orcl
jdbc.username=lee
jdbc.password=tiger
```

# 5·Spring事务管理

- [5·Spring事务管理](#5spring事务管理)
  - [事务管理模板](#事务管理模板)
  - [配置切面对象模板](#配置切面对象模板)
  - [声明式事务注解](#声明式事务注解)
  - [事务传播](#事务传播)
  - [隔离级别](#隔离级别)
  - [回滚规则](#回滚规则)
  - [只读事务](#只读事务)
  - [事务超时](#事务超时)
  - [配置方式实现](#配置方式实现)

## 事务管理模板
Spring提供了事务管理模板，通过AOP环绕通知实现事务管理。支持设置事务传播属性，隔离级别，回滚规则，只读，超时。

## 配置切面对象模板
在常规Spring-SpringMVC-Mybatis框架下，需要通过Spring主配置文件配置事务管理切面对象，对象已被封装。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
  
	<!--开启事务管理注解开发功能-->
  <tx:annotation-driven transaction-manager="transactionManager"/>
  
  <!--配置事务管理切面对象-->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--需要在spring配置文件中配置的数据库连接作为参数-->
    <property name="dataSource" ref="dataSource"/>
  </bean>
</beans>
```

## 声明式事务注解
在需要使用事务的方法上使用`@Transactional`注解即可。

## 事务传播
事务管理默认每个方法为一个事务，但有时不能以一个方法作为一个事务。当一个事务中包含另一个事务，就会发生事务的传播。
通过`propagation`参数设置事务传播属性。

常用属性：

1. `@Transactional(propagation = Propagation.REQUIRED)`

说明：事务始终传播，当一个事务中包含另一个事务，以当前事务为一个事务。

Spring支持七种事务传播行为。

| 事务传播行为类型 | 说明 |
| --- | --- |
| PROPAGATION_REQUIRED | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS | 支持当前事务，如果当前没有事务，就以非事务方式执行。 |
| PROPAGATION_MANDATORY | 使用当前的事务，如果当前没有事务，就抛出异常。 |
| PROPAGATION_REQUIRES_NEW | 新建事务，如果当前存在事务，把当前事务挂起。 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 |
| PROPAGATION_NEVER | 以非事务方式执行，如果当前存在事务，则抛出异常。 |
| PROPAGATION_NESTED | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

## 隔离级别
多个事务之间数据的隔离级别可能有所不同。
通过`isolation`参数设置事务的隔离级别。

常用属性：
1. `@Transactional(isolation = Isolation.SERIALIZABLE)`
说明：序列化，事务一个一个排队执行，隔离级别最强，效率最低。

Spirng支持五种隔离级别。

| 事务隔离级别 | 说明 |
| --- | --- |
| ISOLATION_DEFAULT | 使用后端数据库默认的隔离级别 |
| ISOLATION_READ_COMMITTED | 只允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 |
| ISOLATION_SERIALIZABLE（序列化） | 事务一个一个排队执行。最高的隔离级别，也是最慢的事务隔离级别，通过完全锁定事务相关的数据库表来实现的 |
| ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 |
| ISOLATION_REPEATABLE_READ | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 |

- MySQL支持所有的隔离级别。
- Oracle除默认级别只支持两种：ISOLATION_READ_COMMITTED，ISOLATION_SERIALIZABLE.

## 回滚规则
Spring事务管理默认遇到RuntimeException和Error时才会回滚，但有时事务并不需要回滚。
通过`rollbackFor`参数设置回滚规则。

常用属性：
1. `@Transactional(rollbackFor = Exception.class)`
说明：当程序出现异常时就回滚。

2. `@Transactional(noRollbackFor = ArithmeticException.class)`
说明：当程序出现算术异常时不回滚。

## 只读事务
如果事务只对数据库进行查询，可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，可以给数据库应用它认为合适的优化措施的一个机会。
通过`readOnly`参数设置只读属性。

常用属性：
1. `@Transactional(readOnly = true)`
说明：事务开启只读。

## 事务超时
为了使应用程序更好的运行，事务不能运行太长的时间。事务涉及对后端数据库的锁定，长时间的事务会不必要地占用数据库资源。
事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。
通过`timeout`参数设置超时时间。

常用属性：
1. `@Transactional(timeout = 30)`
说明：事务超时时间30秒。

## 配置方式实现
通过配置文件实现批量事务管理。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置事务管理切面对象-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--需要在spring配置文件中配置的数据库连接作为参数-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--基于配置文件实现事务管理-->
    <!--配置事务属性-->
    <tx:advice transaction-manager="transactionManager" id="myAdvice1">
        <tx:attributes>
            <!--指定哪些方法使用哪些事务属性-->
            <tx:method name="query*" read-only="true"/>
            <tx:method name="add*" isolation="SERIALIZABLE"/>
            <tx:method name="delete*" isolation="SERIALIZABLE"/>
            <tx:method name="update*" isolation="SERIALIZABLE"/>
            <!--除了指定的方法的事务属性外,设置其他没有指定的方法使用默认事务属性,必须写在最后-->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    <!--使用AOP技术,把以上事务属性加到指定的方法上,通过切点表达式指定-->
    <aop:config>
        <!--指定切点表达式-->
        <aop:pointcut id="myPointCut1" expression="execution(* com.java.service.serviceImpl..*.*(..))"/>
        <!--将切点表达式与事务属性进行对应-->
        <aop:advisor advice-ref="myAdvice1" pointcut-ref="myPointCut1"/>
    </aop:config>
</beans>
```

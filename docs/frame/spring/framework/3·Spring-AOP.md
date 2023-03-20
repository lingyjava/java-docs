# 3·Spring-AOP

- [3·Spring-AOP](#3spring-aop)
  - [什么是AOP？](#什么是aop)
  - [动态代理](#动态代理)
  - [应用场景](#应用场景)
  - [切面编程](#切面编程)
  - [切点表达式](#切点表达式)
  - [通知类型](#通知类型)
  - [注解方式实现](#注解方式实现)
  - [配置文件方式实现](#配置文件方式实现)

## 什么是AOP？
面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。
Spring AOP的实现方式：动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

## 动态代理
SpringAOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。

- JDK动态代理：代理对象和目标对象实现同一个接口。
- CGLIB动态代理：代理对象继承目标对象。

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。
如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

## 应用场景
- 日志记录
- 性能统计
- 安全控制
- 权限管理
- 事务处理
- 异常处理
- 资源池管理

## 切面编程
面向切面编程，用来解决动态代理，底层根据具体情况自动选择使用jdk动态代理或cglib动态代理。

- 切面对象：新增业务逻辑封装成的对象，对象中的方法都需要在原有业务逻辑上新增业务逻辑。
- 代理对象：向目标对象应用通知后AOP自动生成的对象。
- 连接点：可以插入切面对象的特殊位置，每个方法都有很多连接点。连接点由两个信息确定，方法表市的程序执行点，相对点表示的方位。
- 切点：每个类都拥有多个连接点，AOP通过切点定位到特定的连接点。
- 类比：连接点相当于数据库中的记录，切点相当于查询条件。一个切点可以匹配多个连接点。

![图10-切面编程](/docs/images/图10-切面编程.jpeg)

## 切点表达式
- 简单示例：`execution(* com.java.service..*.*(..))`

意义：选择service包及其子包下任意类任意方法。
第一个`*`代表任意返回值类型。
第二个`*`代表所有类。
第三个`*`代表所有方法。
第一个`..`代表包及其子包。
第二个`..`代表任意参数。

**模糊查询：**

1. `execution(* com.java.service..*.add*(..))`

意义：选择service包及其子包下任意类方法名以add开头方法。

2. `execution(* com.java.service..*.*Emp*(..))`

意义：选择service包及其子包下任意类方法名包括Emp的方法。

**逻辑运算符：**

1. `execution(* com.java.service..*.*Emp*(..))`

意义：选择service包及其子包下任意类方法名不包括Emp的方法。

## 通知类型
1. 前置通知
注解：@Before()
通知切面方法在目标方法执行前执行。

2. 后置通知
注解：@After()
通知切面方法在目标方法执行后执行。
无论目标方法是否出现异常，都会执行。

3. 返回通知
注解：@AfterReturning
通知切面方法在目标方法执行后执行。
只在方法正确返回结果时执行。
可以获取目标方法的返回值，如果方法没有返回值，获取到null。

4. 异常通知
注解：@AfterThrowing
通知切面方法在目标方法发生异常时执行。
可以获取目标方法出现的异常类型。

5. 环绕通知
注解：@Around
前四个通知的集合。
可以拿到目标方法的返回值，并且修改返回值。
可以控制目标方法是否继续执行。
切面方法必须有一个Object类型的返回值。
如果没有执行proceed()方法会导致，切面方法执行了，但目标方法没执行。

## 注解方式实现
通过使用注解的方式实现动态代理。

1. 在Spring主配置文件中开启AOP注解。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd 
                           http://www.springframework.org/schema/aop 
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
  
  <!--导入aop命名空间,开启aop注解开发功能-->
  <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

2. 编写切面对象。
```java
package com.java.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import java.util.Date;

// 切面必须放到IOC容器
@Component
// 表示这是一个切面对象
@Aspect
public class TestDateAspect {
    
    private Date methodStart;
    
    // 切点表达式,选择目标对象
    // 前置通知,在方法前执行的通知
    @Before("execution(* com.java.service.serviceImpl.CarServiceImpl.queryByFy(..))")
    //JoinPoint jp可选参数
    public void before(JoinPoint jp){
        
        //jp.getSignature()获取方法签名
        methodStart = new Date();
    }
    
    //后置通知,在方法后执行的通知
    @After("execution(* com.java.service.serviceImpl.CarServiceImpl.queryByFy(..))")
    public void after(JoinPoint jp){
        Date methodStop = new Date();
        System.out.println(jp.getSignature()+"方法执行花费时间:"+(methodStop.getTime()-methodStart.getTime())+"毫秒");
    }
}
```

3. 目标对象，目标方法。
```java
public List<Car> queryByFy(FenYeUtil fy) {
    //业务逻辑
    return cars;
}
```

## 配置文件方式实现
使用配置文件的方式来实现动态代理，通过配置文件来配置切面对象。

1. 编写切面对象（同注解方式实现）。
2. 编写AOP配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
  
  <aop:config>
    <!--配置切点表达式-->
    <aop:pointcut id="mypointcut1" expression="execution(* com.java.service.serviceImpl.CarServiceImpl.queryByFy(..))"/>
    <aop:pointcut id="mypointcut2" expression="execution(* com.java.service..*.*(..))"/>
    
    <!--配置切面对象-->
    <aop:aspect ref="testDateAspect">
      <!--配置后置通知,method:方法名,pointcut-ref:选择切点表达式-->
      <aop:after method="after" pointcut-ref="mypointcut2"/>
      <aop:before method="before" pointcut-ref="mypointcut1"/>
      <aop:after-returning method="returning" pointcut-ref="mypointcut1" returning="obj"/>
    </aop:aspect>
  </aop:config>
</beans>
```

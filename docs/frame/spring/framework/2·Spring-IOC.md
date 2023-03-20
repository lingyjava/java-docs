# 2·Spring-IOC

- [2·Spring-IOC](#2spring-ioc)
  - [IOC是什么](#ioc是什么)
  - [IOC特性](#ioc特性)
  - [自动注入](#自动注入)
  - [配置文件注入](#配置文件注入)
  - [DI（依赖注入）](#di依赖注入)
  - [获取Bean实例](#获取bean实例)
  - [常见问题](#常见问题)
    - [找不到Bean](#找不到bean)

## IOC是什么
SpringIOC是一个对象工厂，负责管理对象，无需在程序中new对象，所有的对象从IOC容器中获得。解决类与类之间耦合问题。
Spring自动生成的对象会存放在IOC容器中，如果需要直接从IOC容器中获取即可，Spring主配置文件就相当于IOC容器，主配置文件中的Bean就是需要Spring自动创建的对象。使用Spring管理指定的对象。

## IOC特性
IOC容器特性：
1. 当IOC容器启动时，IOC容器中所有配置的对象就已经完成创建。
2. IOC容器创建的对象默认为单例对象。
3. 可以通过scope属性来改变默认的单例模式。
4. IOC容器创建对象默认调用类的无参构造方法。
5. IOC容器给属性注入值时默认调用其set方法。

## 自动注入
将类注入IOC容器的注解有四种，这些注解功能完全相同，都用来注册成bean，作用是标识MVC分层结构。使用自动注入bean的属性不需要提供get/set方法。

1. @Component：基本注解，标识了一个受spring管理的组件。（不确定层次）
2. @Respository：标识持久层组件（dao层）。
3. @Service：标识服务层组件。
4. @Controller：标识表现层组件。

**设置Bean参数：**
1. @Scope("prototype")：设置bean为原型状态，非单例模式。
2. 通过自动扫描注册的类没有指定id，默认使用类名首字母小写作为id，或在使用注解时通过参数指定id名，如：`@Component("name")`。

## 配置文件注入
通过Spring主配置文件，配置IOC容器，注入Bean实例。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  <!--
    bean:用于配置一个实例(对象)
			id:该bean的唯一标识符
      class:对象对应的类的限定性类名
      scope:prototype  取消该对象单例模式
  -->
	<bean id="stu" class="com.java.pojo.Student" ></bean>
</beans>
```

## DI（依赖注入）
通过Spring主配置文件，注册Bean实例，设置对象相关的属性值。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd">
<beans>  
  <!--给字面值注入属性值,id用来调用-->
	<bean id="stu" class="com.java.pojo.Student" >
    <!--方法1-->
    <property name="age"><value>18</value></property>
    <!--方法2-->
    <property name="name">
      <!--有左括号需要转义-->
      <value><![CDATA[<张三>]]></value>
    </property>
    
    <!--调用有参构造方法-->
    <!--一个标签表示带有一个构造方法参数-->
    <constructor-arg name="name" value="李四"></constructor-arg>
    <constructor-arg name="age" value="20"></constructor-arg>
    <constructor-arg name="id" value="2"></constructor-arg>
  </bean>
  
  <bean id="tea" class="com.java.pojo.Teacher">
    <property name="name" value="lee"></property>
    
    <!--给对象注入属性值-->
    <!--方法1,引用IOC容器中已存在的一个学生对象-->
    <property name="stu" ref="stu"></property>

    <!--方法2,内置bean,无法通过id或类型获取,也无需id属性-->
    <property name="stu">
      <bean class="com.java.pojo.Student">
        <property name="name" value="lee"></property>
        <property name="age" value="18"></property>
      </bean>
    </property>
    
    <!--给List集合注入属性值-->
    <property name="ls">
      <!--相当于该属性是一个List集合-->
      <list>
        <!--方法1,引用bean-->
        <ref bean="stu"></ref>
        <!--方法2,内置bean-->
        <bean class="com.java.pojo.Student">
          <property name="name" value="lee"></property>
        </bean>
      </list>
    </property>
    
    <!--给Map集合注入属性值-->
    <property name="ms">
      <map>
        <!--引用-->
        <entry key="张三" value-ref="stu"></entry>
        <entry key="李四">
          <!--内置bean-->
          <bean class="com.java.pojo.Student">
            <property name="name" value="李四"></property>
          </bean>
        </entry>
      </map>
    </property>
    
    <!--给Date注入-->
    <property name="hireDate">
      <bean factory-bean="dateFormat" factory-method="parse">
        <constructor-arg value="2021-09-13"></constructor-arg>
      </bean>
    </property>
  </bean>

  <!--配置日期解析的工厂bean:SimpleDateFormat-->
  <bean id="dateFormat" class="java.text.SimpleDateFormat">
    <constructor-arg name="pattern" value="yyyy-MM-dd"></constructor-arg>
  </bean>
</beans>
```

## 获取Bean实例
从ApplicationContext应用上下文中读取配置文件，获取Bean实例。
```java
import com.java.pojo.Student;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        //classpath:	类路径,指的是src目录
        ApplicationContext app = new ClassPathXmlApplicationContext("classpath:springIOC.xml");
        
        Student stu = (Student) app.getBean("stu");//bena-->id属性
        stu.setId(1);stu.setAge(18);stu.setName("lee");
        System.out.println(stu);
    }
}
```

## 常见问题

### 找不到Bean

1. 检查SpringBoot项目是否添加注解开启注解扫描。
```java
@ComponentScan(value = "com.ly")
@MapperScan(basePackages = {"com.ly.mapper"})
```

2. 或者是在Spring主配置文件中配置注解扫描。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd">
  <!-- 导入context命名空间,开启自动扫描 -->
  <context:component-scan base-package="com.ly"></context:component-scan>
</beans>
```

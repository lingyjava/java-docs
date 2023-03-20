# 2·SpringBoot常用功能

- [2·SpringBoot常用功能](#2springboot常用功能)
  - [热部署](#热部署)
  - [JSP](#jsp)
  - [Oracle](#oracle)
  - [MySQL](#mysql)
  - [Redis](#redis)
  - [POI](#poi)
    - [Maven依赖](#maven依赖)
    - [使用示例](#使用示例)
  - [图片上传](#图片上传)
  - [Logback](#logback)
  - [定时任务](#定时任务)
    - [Quartz](#quartz)
    - [CRON表达式](#cron表达式)
  - [Filter](#filter)
  - [Session共享](#session共享)
  - [多环境配置](#多环境配置)
    - [主配置文件](#主配置文件)
    - [配置项加密](#配置项加密)
    - [jasypt](#jasypt)
    - [配置项注入静态static与非静态属性](#配置项注入静态static与非静态属性)

## 热部署
- [SpringBoot入门 - 配置热部署devtools工具](https://pdai.tech/md/spring/springboot/springboot-x-hello-devtool.html)

Springboot支持项目热部署，可以在创建项目时选择依赖，也可以手动导入。
1. 导入热部署相关依赖。
2. 配置idea监测到项目修改以后重新部署项目。
设置-->构建、执行、部署-->编译器-->自动构建项目
3. idea高级配置。
设置-->高级设置-->编译器-->即使当前正在运行所开发的应用程序，也允许启动自动生成。

## JSP
SpringBoot不建议使用Jsp，也可以不接受建议。

使用步骤：
1. 导入Springboot解析Jsp页面依赖。
2. 添加配置文件。
3. 程序入口添加注解。
4. 编写controller，service，dao等。
5. 启动SpringbootApplication主方法，相当于启动内置的tomcat服务器。
6. 项目访问，Springboot发布的项目路径默认不带项目名称。

## Oracle
1. 导入oracle数据库依赖。
2. 添加配置文件。

## MySQL
1. 映射文件所在目录：建议放在resources目录下。
2. 添加配置文件。
3. 程序入口添加注解。dao层接口路径。

## Redis
1. 导入redis相关依赖。
2. 在配置文件中配置redis连接相关属性，导入依赖必须配置，否则报错。
3. 在使用redis的服务层注入redisTemplate模板使用。

```java
public class DeptServiceImpl implement DeptService{
    @Autowired
    private DeptDao dd;
    @AutoWired
    @Qualifier("redisTemplate")//模板的kv都是Object类型
    private RedisTemplate redisTemplate rt;
    @Override
    @Transactional//事务管理
    public List<Dept> queryAllDept(){
        ValueOperations vo = rt.opsForValue();
        //先尝试从redis数据库中获取
        List<Dept> depts = (List<Dept>) vo.get("depts");
    	if(depts == null){
            //获取失败,查询数据库
            depts= dd.queryAllDept();
            //存入redis数据库
            vo.set("depts",depts);
    }
}
```

## POI
- [SpringBoot集成文件 - 集成POI之Excel导入导出](https://pdai.tech/md/spring/springboot/springboot-x-file-excel-poi.html#springboot%E9%9B%86%E6%88%90%E6%96%87%E4%BB%B6---%E9%9B%86%E6%88%90poi%E4%B9%8Bexcel%E5%AF%BC%E5%85%A5%E5%AF%BC%E5%87%BA)

Apache POI 是用Java编写的免费开源的跨平台的 Java API，Apache POI提供API给Java程序对Microsoft Office格式档案读和写的功能。POI为“Poor Obfuscation Implementation”的首字母缩写，意为“简洁版的模糊实现”。

### Maven依赖
```xml
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>5.2.2</version>
</dependency>
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi-ooxml</artifactId>
  <version>5.2.2</version>
</dependency>
```

### 使用示例
```java
@GetMapping("/excel/download")
public void download(HttpServletResponse response) {
    try {
        SXSSFWorkbook workbook = userService.generateExcelWorkbook();
        response.reset();
        response.setContentType("application/vnd.ms-excel");
        response.setHeader("Content-disposition",
                "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
        OutputStream os = response.getOutputStream();
        workbook.write(os);
        workbook.dispose();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

## 图片上传
Springboot默认已经完成图片上传的配置。
在Springboot中不建议使用配置文件，图片上传需要创建的对象，不在Spring中配置Bean，写一个配置文件类，直接new对象。

1. 导入依赖。
2. 在config包中写一个配置类。
```java
package com.ly.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author ly
 * @Date: 2021/10/15 13:47
 */
@Configuration//声明这是一个配置文件类
public class FileUploadConfig implements WebMvcConfigurer {

    /**
     * 相当于一个bean
     * */
    @Bean
    public CommonsMultipartResolver multipartResolver(){
        CommonsMultipartResolver cr = new CommonsMultipartResolver();
        cr.setResolveLazily(true);
        cr.setDefaultEncoding("utf-8");
        cr.setMaxUploadSize(1024000);
        return cr;
    }

    /**
     * 配置图片虚拟路径
     * */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/img/**").addResourceLocations("file:d:/Pictures/img/springbootEmp/");
        WebMvcConfigurer.super.addResourceHandlers(registry);
    }
}
```

3. 文件上传服务编写。
```java
package com.ly.service.serviceImpl;

import com.ly.pojo.Emp;
import com.ly.service.FileUploadService;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

@Service
public class FileUploadServiceImpl implements FileUploadService {

    /**
     * 针对emp表的图片上传
     * */
    @Override
    public Emp empFileUpload(Emp emp, CommonsMultipartFile avatar) {

        //获取图片原来的名字
        String oldImgName = avatar.getOriginalFilename();
        if (oldImgName != null && !"".equals(oldImgName)){
            //截取原名字后缀拼一个随机的新名字
            String newImgName = UUID.randomUUID() +oldImgName.substring(oldImgName.lastIndexOf("."));
            //上传框数据
            File imgFile = new File("d:\\Pictures\\img\\springbootEmp\\"+newImgName);
            try {
                //把获取到的上传框数据复制到我们指定的新文件中
                avatar.transferTo(imgFile);
            } catch (IOException e) {
                e.printStackTrace();
            }
            emp.setImg("/img/"+newImgName);
        }
        return emp;
    }
}
```

## Logback
Springboot默认继承logback日志框架。

1. 在resouces目录下编写一个logback-spring.xml的日志配置文件。
```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
  <!-- appender是configuration的子节点，是负责写日志的组件。 -->
  <!-- ConsoleAppender：把日志输出到控制台 -->
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- 默认情况下，每个日志事件都会立即刷新到基础输出流。 这种默认方法更安全，因为如果应用程序在没有正确关闭appender的情况下退出，则日志事件不会丢失。
         但是，为了显着增加日志记录吞吐量，您可能希望将immediateFlush属性设置为false -->
    <!--<immediateFlush>true</immediateFlush>-->
    <encoder>
      <!-- %37():如果字符没有37个字符长度,则左侧用空格补齐 -->
      <!-- %-37():如果字符没有37个字符长度,则右侧用空格补齐 -->
      <!-- %15.15():如果记录的线程字符长度小于15(第一个)则用空格在左侧补齐,如果字符长度大于15(第二个),则从开头开始截断多余的字符 -->
      <!-- %-40.40():如果记录的logger字符长度小于40(第一个)则用空格在右侧补齐,如果字符长度大于40(第二个),则从开头开始截断多余的字符 -->
      <!-- %msg：日志打印详情 -->
      <!-- %n:换行符 -->
      <!-- %highlight():转换说明符以粗体红色显示其级别为ERROR的事件，红色为WARN，BLUE为INFO，以及其他级别的默认颜色。 -->
      <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) --- [%15.15(%thread)] %cyan(%-40.40(%logger{40})) : %msg%n</pattern>
      <!-- 控制台也要使用UTF-8，不要使用GBK，否则会中文乱码 -->
      <charset>UTF-8</charset>
    </encoder>
  </appender>

  <!-- info 日志-->
  <!-- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
  <!-- 以下的大概意思是：1.先按日期存日志，日期变了，将前一天的日志文件名重命名为XXX%日期%索引，新的日志仍然是project_info.log -->
  <!--             2.如果日期没有发生变化，但是当前日志的文件大小超过10MB时，对当前日志进行分割 重命名-->
  <appender name="info_log" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--日志文件路径和名称-->
    <File>D:/log/springboot1-info.log</File>
    <!--是否追加到文件末尾,默认为true-->
    <append>true</append>
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>DENY</onMatch><!-- 如果命中ERROR就禁止这条日志 -->
      <onMismatch>ACCEPT</onMismatch><!-- 如果没有命中就使用这条规则 -->
    </filter>
    <!--有两个与RollingFileAppender交互的重要子组件。 第一个RollingFileAppender子组件，即RollingPolicy:负责执行翻转所需的操作。
 RollingFileAppender的第二个子组件，即TriggeringPolicy:将确定是否以及何时发生翻转。 因此，RollingPolicy负责什么和TriggeringPolicy负责什么时候.
     作为任何用途，RollingFileAppender必须同时设置RollingPolicy和TriggeringPolicy,但是，如果其RollingPolicy也实现了TriggeringPolicy接口，则只需要显式指定前者。-->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- 日志文件的名字会根据fileNamePattern的值，每隔一段时间改变一次 -->
      <!-- 文件名：logs/project_info.2017-12-05.0.log -->
      <!-- 注意：SizeAndTimeBasedRollingPolicy中 ％i和％d令牌都是强制性的，必须存在，要不会报错 -->
      <fileNamePattern>logs/project_info.%d.%i.log</fileNamePattern>
      <!-- 每产生一个日志文件，该日志文件的保存期限为30天, ps:maxHistory的单位是根据fileNamePattern中的翻转策略自动推算出来的,例如上面选用了yyyy-MM-dd,则单位为天
    如果上面选用了yyyy-MM,则单位为月,另外上面的单位默认为yyyy-MM-dd-->
      <maxHistory>30</maxHistory>
      <!-- 每个日志文件到10mb的时候开始切分，最多保留30天，但最大到20GB，哪怕没到30天也要删除多余的日志 -->
      <totalSizeCap>20GB</totalSizeCap>
      <!-- maxFileSize:这是活动文件的大小，默认值是10MB，测试时可改成5KB看效果 -->
      <maxFileSize>10MB</maxFileSize>
    </rollingPolicy>
    <!--编码器-->
    <encoder>
      <!-- pattern节点，用来设置日志的输入格式 ps:日志文件中没有设置颜色,否则颜色部分会有ESC[0:39em等乱码-->
      <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level --- [%15.15(%thread)] %-40.40(%logger{40}) : %msg%n</pattern>
      <!-- 记录日志的编码:此处设置字符集 - -->
      <charset>UTF-8</charset>
    </encoder>
  </appender>

  <!-- error 日志-->
  <!-- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
  <!-- 以下的大概意思是：1.先按日期存日志，日期变了，将前一天的日志文件名重命名为XXX%日期%索引，新的日志仍然是project_error.log -->
  <!--             2.如果日期没有发生变化，但是当前日志的文件大小超过10MB时，对当前日志进行分割 重命名-->
  <appender name="error_log" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!--日志文件路径和名称-->
    <File>D:/log/springboot1-error.log</File>
    <!--是否追加到文件末尾,默认为true-->
    <append>true</append>
    <!-- ThresholdFilter过滤低于指定阈值的事件。 对于等于或高于阈值的事件，ThresholdFilter将在调用其decision（）方法时响应NEUTRAL。 但是，将拒绝级别低于阈值的事件 -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>ERROR</level><!-- 低于ERROR级别的日志（debug,info）将被拒绝，等于或者高于ERROR的级别将相应NEUTRAL -->
    </filter>
    <!--有两个与RollingFileAppender交互的重要子组件。 第一个RollingFileAppender子组件，即RollingPolicy:负责执行翻转所需的操作。
RollingFileAppender的第二个子组件，即TriggeringPolicy:将确定是否以及何时发生翻转。 因此，RollingPolicy负责什么和TriggeringPolicy负责什么时候.
                                                          作为任何用途，RollingFileAppender必须同时设置RollingPolicy和TriggeringPolicy,但是，如果其RollingPolicy也实现了TriggeringPolicy接口，则只需要显式指定前者。-->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- 活动文件的名字会根据fileNamePattern的值，每隔一段时间改变一次 -->
      <!-- 文件名：logs/project_error.2017-12-05.0.log -->
      <!-- 注意：SizeAndTimeBasedRollingPolicy中 ％i和％d令牌都是强制性的，必须存在，要不会报错 -->
      <fileNamePattern>logs/project_error.%d.%i.log</fileNamePattern>
      <!-- 每产生一个日志文件，该日志文件的保存期限为30天, ps:maxHistory的单位是根据fileNamePattern中的翻转策略自动推算出来的,例如上面选用了yyyy-MM-dd,则单位为天
    如果上面选用了yyyy-MM,则单位为月,另外上面的单位默认为yyyy-MM-dd-->
      <maxHistory>30</maxHistory>
      <!-- 每个日志文件到10mb的时候开始切分，最多保留30天，但最大到20GB，哪怕没到30天也要删除多余的日志 -->
      <totalSizeCap>20GB</totalSizeCap>
      <!-- maxFileSize:这是活动文件的大小，默认值是10MB，测试时可改成5KB看效果 -->
      <maxFileSize>10MB</maxFileSize>
    </rollingPolicy>
    <!--编码器-->
    <encoder>
      <!-- pattern节点，用来设置日志的输入格式 ps:日志文件中没有设置颜色,否则颜色部分会有ESC[0:39em等乱码-->
      <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level --- [%15.15(%thread)] %-40.40(%logger{40}) : %msg%n</pattern>
      <!-- 记录日志的编码:此处设置字符集 - -->
      <charset>UTF-8</charset>
    </encoder>
  </appender>

  <!--给定记录器的每个启用的日志记录请求都将转发到该记录器中的所有appender以及层次结构中较高的appender（不用在意level值）。
        换句话说，appender是从记录器层次结构中附加地继承的。
        例如，如果将控制台appender添加到根记录器，则所有启用的日志记录请求将至少在控制台上打印。
        如果另外将文件追加器添加到记录器（例如L），则对L和L'子项启用的记录请求将打印在文件和控制台上。
                                                          通过将记录器的additivity标志设置为false，可以覆盖此默认行为，以便不再添加appender累积-->
  <!-- configuration中最多允许一个root，别的logger如果没有设置级别则从父级别root继承 -->
  <root level="DEBUG">
    <appender-ref ref="info_log" />
    <appender-ref ref="error_log" />
    <appender-ref ref="STDOUT" />
  </root>

  <!-- 指定项目中某个包，当有日志操作行为时的日志记录级别 -->
  <!-- 级别依次为【从高到低】：FATAL > ERROR > WARN > INFO > DEBUG > TRACE  -->
  <logger name="com.sailing.springbootmybatis" level="DEBUG">
    <appender-ref ref="info_log" />
    <appender-ref ref="error_log" />
  </logger>

  <!-- 利用logback输入mybatis的sql日志，
        注意：如果不加 additivity="false" 则此logger会将输出转发到自身以及祖先的logger中，就会出现日志文件中sql重复打印-->
  <logger name="com.sailing.springbootmybatis.mapper" level="DEBUG" additivity="false">
    <appender-ref ref="info_log" />
    <appender-ref ref="error_log" />
    <appender-ref ref="STDOUT" />
  </logger>
  <!-- additivity=false代表禁止默认累计的行为，即com.atomikos中的日志只会记录到日志文件中，不会输出层次级别更高的任何appender-->
  <logger name="com.atomikos" level="INFO" additivity="false">
    <appender-ref ref="info_log" />
    <appender-ref ref="error_log" />
    <appender-ref ref="STDOUT" />
  </logger>
</configuration>
```

2. 在配置文件中指定日志文件的路径。

## 定时任务

### Quartz
1. 导入Quartz相关依赖。
2. 在程序入口通过注解开启任务调度。
3. 编写任务调度执行的业务逻辑,并且通过注解指定该任务执行的规则。
```java
package com.ly.qz;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * @author ly
 * @Date: 2021/10/16 13:42
 */
@Component
public class TestQZ {

    @Scheduled(fixedRate = 5000)
    public void getDate(){
        System.out.println("当前系统时间"+new Date());
    }
}
```

### CRON表达式
[Cron表达式的语法及详细用法](https://blog.csdn.net/wh13267207590/article/details/80095128)

任务调度常用的规则：
`@Scheduled(fixedRate = 5000)` ：上一次开始执行时间点之后5秒再执行
`@Scheduled(fixedDelay = 5000)` ：上一次执行完毕时间点之后5秒再执行
`@Scheduled(initialDelay=1000, fixedRate=5000)` ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次`

自定义规则表达式：
`@Scheduled(cron=" /5 ")` ：通过cron表达式定义规则

## Filter
在程序入口使用注解配置filter所在的包目录（声明servlet相关的api的目录），在对应包中编写过滤器。

```java
package com.ly.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

/**
 * @author ly
 * @Date: 2021/10/15 14:03
 */
//使用注解声明过滤的请求
@WebFilter(urlPatterns = {"/user/*","/emp/*"})
public class UserFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器创建");;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("进入过滤器");
        //放过请求
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");;
    }
}
```

## Session共享
1. 导入session共享的依赖。
2. 在程序入口通过注解开启session共享。

## 多环境配置
项目开发中会有多个应用环境，开发环境、测试环境、生产环境，各个环境的配置略有不同，可以创建多份配置文件，由主配置文件来控制读取那个子配置。

创建spring boot项目后可以同时创建多个`.properties`文件，只要符合它要求的格式即可。
文件格式：`application-{profile}.properties`{profile}是变量用于自定义配置文件名称

例如：
```properties
spring.application.name=project-dev
server.port=8080
```
```properties
spring.application.name=project-test
server.port=8090
```
```properties
spring.application.name=project-pre
server.port=9080
```
```properties
spring.application.name=project-prod
server.port=9090
```

### 主配置文件
在主配置文件选择子配置文件。
```properties
# 具体使用那个配置文件的标识名称
# 格式：application-{profile}.properties
# {profile}是变量用于自定义配置文件名称
spring.profiles.active=dev
```

### 配置项加密
properties文件中有很多敏感信息，如：数据库连接、缓存服务器连接等等，这些用户名密码都应该是外部不可见的，所以最好将其加密后存储。

### jasypt
可以使用jasypt来进行加解密，其使用的是PBEWithMD5AndDES加密方式，所以每次加密出来的结果都不一样，所以很适合对数据进行加密。
```xml
<!-- 配置文件项加密 -->
<dependency>
  <groupId>com.github.ulisesbocchio</groupId>
  <artifactId>jasypt-spring-boot-starter</artifactId>
  <version>2.1.0</version>
</dependency>
```
在`application.properties`文件中增加配置项，需要jasypt来解密的密文需要用“ENC(......)”括起来。
```properties
spring.application.name=project-prod
server.port=9090
# 配置文件项加解密密码，此处注释，而放在代码中（放在代码中使加密密钥和密文分开）
#jasypt.encryptor.password=112233
# 模拟数据库连接帐号密码
spring.datasource.username=ENC(nm3F96GtUIwZUHzsP0Mp1A==)
spring.datasource.password=ENC(lmn7lAlePy1hZu505WO0xQ==)
```
程序启动类，默认jasypt的密钥是放在配置文件中但这样会导致密文和密钥都在配置文件中，所以我把密钥放在程序中
```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        //设置配置文件项加密密钥（放在这里使加密密钥和密文分开）
        System.setProperty("jasypt.encryptor.password", "112233");
        SpringApplication.run(App.class, args);
    }
}
```
使用注解的方式来注入配置文件中的配置项。
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class SysConfig {

    @Value("${spring.datasource.username}")
    private String dbUsername;

    @Value("${spring.datasource.password}")
    private String dbPassword;
    //省略get set方法
}
```

### 配置项注入静态static与非静态属性
很多编码需求需要使用.properties文件中自定义的配置项，传统使用Properties对象来操作，类似如下代码，这种方式太过灵活，不想使用的配置项可能也会被提取出来，而且当不想使用jar包内的配置文件，而是利用优先级使用外部的，这种直接读取的方式就很不方便，所以推荐使用@Value的方式来使用。
```java
public class SysConfigUtil {
    private static Properties props;
    static {
        try {
            // TODO：读取用户配置
            Resource resource = new ClassPathResource("/application.properties");
            props = PropertiesLoaderUtils.loadProperties(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static String getProperty(String key) {
        return props == null ? null : props.getProperty(key);
    }
｝
```
使用@Value来注入想让程序使用的配置项，而不想让程序使用的就不注入，这样来使配置项可控。
1、在.properties文件中增加自定义配置项
```java
spring.application.name=tyh-demo-prop
server.port=10001
# 配置文件项加解密密码，此处注释，而放在代码中（放在代码中使加密密钥和密文分开）
#jasypt.encryptor.password=112233
# 模拟数据库连接帐号密码
spring.datasource.username=ENC(nm3F96GtUIwZUHzsP0Mp1A==)
spring.datasource.password=ENC(lmn7lAlePy1hZu505WO0xQ==)
# 模拟其他自定义配置项
#tyh.url.web.admin=http://www.admin.com
tyh.url.web.agent=http://www.agent.com
```
2、@Value注入可以给静态属性也可以给非静态属性，具体根据使用场景自行决定，如果配置项可能不存在也可以设置默认值，避免程序无法启动。
```java
@Component
public class SysConfig {

    @Value("${spring.datasource.username}")
    private String dbUsername;

    @Value("${spring.datasource.password}")
    private String dbPassword;

    /*
    非静态属性注入（注入属性）
     */
    //@Value的参数代表配置项的key，如果没有启动会报错，加上“:”为其设置默认值即可解决冒号后面的就是默认值内容，也可以直接:冒号后面空白就是空
    @Value("${tyh.url.web.admin:www.abc.com}")
    private String urlAdmin;

    //###自己创建get/set方法###

    /*
    静态属性注入（注入set()方法）
     */
    //使用@Component把当前类当作组件启动时注入该静态属性值，静态属性注入set()方法
    public static String urlAgent;
    @Value("${tyh.url.web.agent:}")
    private void setUrlAgent(String urlAgent) {
        SysConfig.urlAgent = urlAgent;
    }
}
```
3、使用时非静态属性使用Autowired注入，静态属性直接取值。
```java
//非静态属性注入取值（必须使用Autowired注入）
@Autowired
SysConfig sysConfig;

public void test() {
    //静态属性注入取值（直接获取）
    String str = SysConfig.urlAgent;
}
```

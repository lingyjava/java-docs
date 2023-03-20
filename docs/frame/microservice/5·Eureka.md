# 5·Eureka

- [5·Eureka](#5eureka)
  - [Eureka是什么](#eureka是什么)
  - [搭建项目](#搭建项目)
    - [客户端搭建](#客户端搭建)

## Eureka是什么
微服务注册中心，AP设计，保证了可用性，牺牲了一致性，无主从节点，一个节点挂了，自动切换其他节点可以使用，服务节点间的数据可能不一致，去中心化。springCloud推荐。对注册中心服务可用性要求较高时使用，服务高可用。
Eureka在设计时优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。Eureka的客户端在向某个Eureka注册时如果发现连接失败，则会自动切换到其他节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，但是查到的信息可能不是最新的(不保证一致性)。

Eureka还有一种自我保护机制，如果15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时：
1. Eureka不再从注册列表中移除因长时间没收到心跳而应该过期的服务。
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上(保证当前节点依然可用)。
3. 当网络稳定时，当前实例新的注册信息会被同步到其他节点中。
因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样整个注册服务瘫痪。

## 搭建项目
1. 创建Maven主项目（仅提供公共依赖，作为目录）。
2. 创建子模块，springBoot项目。
3. 添加Eureka或Zookeeper服务相关的依赖。
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

4. 在启动类上添加注解@EnableEurekaServer声明该项目为Eureka注册中心服务器。
5. 配置application.yml
```yaml
# 服务配置
server:
  port: 8050 # 注册中心端口号

# eureka相关配置
eureka:
  instance:
    hostname: 127.0.0.1 # 注册中心服务器名称
  client:
    register-with-eureka: false # 该注册中心是否接收http请求（没有控制层）
    fetch-registry: false # 是否拉取服务信息
    service-url:
      defaultZone: http://127.0.0.1:8050/eureka/ # 客户端要连接注册中心使用该地址
```

5. 访问注册中心页面。
[http://127.0.0.1:8050/](http://127.0.0.1:8050/)

### 客户端搭建
1. 创建子模块，springBoot项目。
2. 添加服务注册和发现依赖。
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

3. 在启动类上添加注解@EnableEurekaClient声明该项目是被Eureka注册中心接管的一个项目。
4. 在application.yml中添加配置文件，指定注册中心的路径。
```yaml
# 配置服务
server:
  port: 8081  # 服务端口号
  
# 连接注册中心的配置
eureka:
  client:
    service-url: 
      defaultZone: http://127.0.0.1:8050/eureka/
      
# 该服务在注册中心的名称
spring:
  application:
    name: goods-server
  # freemarker
  freemarker:
    suffix: .html
    cache: false
  # oracle
  datasource:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    url: jdbc:oracle:thin:@127.0.0.1:1521:orcl
    username: ly
    password: tiger
  # redis
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379

# mybatis
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.ly.pojo
```

5. stock服务和order服务相关的业务逻辑。

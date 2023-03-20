# 4·Gateway

- [4·Gateway](#4gateway)
  - [官方文档](#官方文档)
  - [Gateway是什么？](#gateway是什么)
  - [搭建网关服务](#搭建网关服务)
  - [路由断言工厂](#路由断言工厂)
  - [路由过滤器](#路由过滤器)
    - [默认过滤器](#默认过滤器)
  - [全局过滤器](#全局过滤器)

## 官方文档
[spring cloud gateway 2.2.2 中文文档](https://www.icepear.cn/2020/05/14/springcloud/springcloudgateway/)

## Gateway是什么？
在SpringCloud中网关的实现有Zuul和Gateway，
Zuul是基于Servlet的实现，属于阻塞式编程。
Gateway基于Spring5中提供的webflux，属于响应式编程的实现，具备更好的性能。

显而易见Gateway是分布式架构中的网关中间件，主要用于：
- 身份认证和权限校验
- 服务路由和负载均衡
- 请求限流

## 搭建网关服务
搭建步骤：
1. 创建一个module，引入SpringCloudGateway的依赖和nacos的服务发现依赖。
```xml
<!--SpringCloudGateway-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
  <version>2.2.10.RELEASE</version>
</dependency>
<!--nacos服务发现-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  <version>2.2.7.RELEASE</version>
</dependency>
```

2. 编写启动类。
```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
}
```

3. 编写基本配置文件。
```yaml
server:
  port: 10010  #网关端口
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
      - id: user-service #路由id,自定义,唯一即可
        uri: lb://userservice #路由的目标地址,lb代表负载均衡,后面是服务名称
        predicates: #路由断言,判断请求是否符合路由规则的条件
        - Path=/user/** #按照路径匹配,只要以/user/开头就符合要求
```

## 路由断言工厂
网关路由可以配置的内容包括：
- 路由id：路由的唯一标识
- uri：路由目的地，支持lb和http
- predicates：路由断言，判断请求是否符合要求，符合则转发到路由目的地
- filters：路由过滤器，处理请求或响应

在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断条件。
- After：是某个时间点后的请求`-After=2022-01-01T01:00:59.789-07:00[America/Denver]`
- Before：是某个时间点前的请求`-Before=-After=2022-01-01T01:00:59.433+08:00[America/Denver]`
- Between：是某两个时间点之前的请求`-Between=2022-01-01T01:00:59.789-07:00[America/Denver],2030-01-01T01:00:59.433+08:00[America/Denver]`
- Cookie：请求必须包含某些cookie`-Cookie=chocolate,ch.p`
- Header：请求必须包含某些Header`-Header=X-Request-Id,\d+`
- Host：请求必须是访问某个host（域名）`-Host=**.somehost.org,**.anotherhost.org`
- Method：请求方式必须是指定方式`-Method=GET,POST`
- Path：请求路径必须符合指定规则`-Path=/red/{segment},/blue/**`
- Query：请求参数必须包含指定参数`-Query=name,Jake`
- RemoteAddr：请求者的ip必须是指定范围`-RemoteAddr=192.168.1.1/24`
- Weight：权重处理
```yaml
server:
  port: 10010  #网关端口
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
      - id: user-service #路由id,自定义,唯一即可
        uri: lb://userservice #路由的目标地址,lb代表负载均衡,后面是服务名称
        predicates: #路由断言,判断请求是否符合路由规则的条件
        - Path=/user/** #按照路径匹配,只要以/user/开头就符合要求
        -After=2022-01-01T01:00:59.789-07:00[America/Denver]
```

## 路由过滤器
GatewayFilter是网关提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理。
Spring提供了31种路由过滤器，详见文档。
- AddRequestHeader：给当前请求添加一个请求头
- RemoveRequestHeader：移除请求中的一个请求头
- AddResponseHeader：给响应结果中添加一个请求头
- RemoveResponseHeader：从响应结果中移除一个响应头
- RequestRateLimiter：限制请求的流量
```yaml
server:
  port: 10010  #网关端口
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
      - id: user-service #路由id,自定义,唯一即可
        uri: lb://userservice #路由的目标地址,lb代表负载均衡,后面是服务名称
        predicates: #路由断言,判断请求是否符合路由规则的条件
        - Path=/user/** #按照路径匹配,只要以/user/开头就符合要求
        -After=2022-01-01T01:00:59.789-07:00[America/Denver]
        filters: #过滤器
        -AddRequestHeader=meg, ly!
```

### 默认过滤器
```yaml
server:
  port: 10010  #网关端口
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
      - id: user-service #路由id,自定义,唯一即可
        uri: lb://userservice #路由的目标地址,lb代表负载均衡,后面是服务名称
        predicates: #路由断言,判断请求是否符合路由规则的条件
        - Path=/user/** #按照路径匹配,只要以/user/开头就符合要求
        - After=2022-01-01T01:00:59.789-07:00[America/Denver]
        filters: #过滤器
        -AddRequestHeader=meg, ly!
      defalut-filters: #默认过滤器,对所有的路由请求生效
        - AddRequestHeader=meg, ly!
```

## 全局过滤器
GlobalFilter的作用是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。
GatewayFilter通过配置定义，处理逻辑固定，GlobalFilter的逻辑需要自己实现，可以自定义。
定义的方式是实现GlobalFilter接口。
```java
public interface GlabalFilter{
    
    /**
     * 处理当前请求,通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
     * @param exchange 请求上下文,里面可以获取Request、Response等信息
     * @param chain 用来把请求委托给下一个过滤器
     * @return {@code Mono<Void>} 返回标识当前过滤器业务结束
     */
    Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```

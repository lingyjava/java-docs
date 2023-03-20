# 6·Nacos

- [6·Nacos](#6nacos)
  - [Nacos是什么](#nacos是什么)
  - [搭建Nacos](#搭建nacos)
  - [服务注册](#服务注册)
  - [服务集群](#服务集群)
  - [负载均衡](#负载均衡)

## Nacos是什么
阿里巴巴开源的注册中心Nacos，现作为SpringCloud的注册中心组件。

## 搭建Nacos
1. 下载解压Nacos-server
2. 修改conf文件
3. 通过命令行启动服务

## 服务注册
1. 为服务添加依赖。
```xml
<!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-discovery -->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  <version>2.2.5.RELEASE</version>
</dependency>
```

2. 配置Nacos的地址。
```properties
spring.cloud.nacos.server-addr:localhost:8848
```

## 服务集群
配置服务集群，服务优先访问本地集群。

1. 配置Nacos集群。
```properties
#名称自定义
spring.cloud.nacos.discovery.cluster-name:SZ 
```

## 负载均衡
设置负载均衡的规则。集群模式下，设置随机访问将不会优先访问本地集群。

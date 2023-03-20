# 9·Hystrix

- [9·Hystrix](#9hystrix)
  - [Hystrix是什么](#hystrix是什么)
  - [配置](#配置)

## Hystrix是什么
网飞开源的服务熔断组件。
[Hystrix](https://github.com/Netflix/Hystrix)

在分布式环境中，服务间的依赖有时会不可避免地失败。某个服务可能因为不可抗力的因素挂掉一会，这时调用这个挂掉的服务的方法的另一个服务可能就会产生故障。可能因为等不到响应一直等待，可能因为调用失败反复调用，那么容器中的线程数量则会持续增加直致CPU资源耗尽到100%，整个服务对外不可用，集群环境下就是雪崩。导致了服务的级联故障。
Hystrix 熔断器，通过线程池隔离、信号量隔离、熔断、降级回退来控制分布式服务之间的交互。隔离服务之间的访问点、阻止它们之间的级联故障，并提供回退选项来实现这一点，提高系统的整体弹性。
它是系统负载过高，突发流量或者网络等各种异常情况介绍，常用的解决方案。在一个分布式系统里，一个服务依赖多个服务，可能存在某个服务调用失败，比如超时、异常等，如何能够保证在一个依赖出问题的情况下，不会导致整体服务失败。
什么是熔断：一般是某个服务故障或者是异常引起的，类似现实世界中的‘保险丝’，当某个异常条件被触发，直接熔断整个服务，而不是一直等到此服务超时，为了防止防止整个系统的故障，而采用了一些保护措施。
什么是降级：服务器当压力剧增的时候，根据当前业务情况及流量，对一些服务和页面进行有策略的降级。以此缓解服务器资源的的压力，以保证核心业务的正常运行，同时也保持了客户和大部分客户的得到正确的相应。(主逻辑失败采用备用逻辑)

## 配置
1. 设置熔断器超时
```yaml
#熔断器超时
hystrix: 
  command: 
    default:
      execution:
        timeout: 
          enabled: true
        isolation: 
          thread: 
            		timeoutInMilliseconds: 5000
```

2. 导入依赖。
```xml
<!--熔断器Hystrix相关依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

3. 服务消费者在启动类上添加注解@EnableCircuitBreaker
4. 开启feign支持hystrix
```yaml
# 开启feign支持hystrix
feign:
	 hystrix:
		enabled: true
```

5. 为API增加一个实现类。
```java
package com.ly.api.apiImpl;

import com.ly.api.PointsAPI;
import com.ly.pojo.Points;
import java.util.HashMap;
import java.util.Map;

public class PointServiceAPIFallbackImpl implements PointsAPI {
    @Override
    public Map<String, Object> addPoints(Points points) {
        //回退处理
        System.out.println("回退业务逻辑");
        Map<String,Object> map = new HashMap<>();
        map.put("error","error");   
        return map;
    }
}
```

6. 为接口的注解添加参数@FeignClient(name = "points-server",fallback = PointServiceAPIFallbackImpl.class)
7. 可以调用短信接口发送短信提醒开发人员对应服务出现问题，或者通过redis或本地日志的形式把对应的数据做托底处理。

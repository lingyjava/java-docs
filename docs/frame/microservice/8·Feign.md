# 8·Feign

- [8·Feign](#8feign)
  - [Feign是什么](#feign是什么)
  - [服务拆分及远程调用](#服务拆分及远程调用)
  - [远程调用的问题](#远程调用的问题)
  - [Feign](#feign)
  - [搭建Feign](#搭建feign)

## Feign是什么
微服务间远程服务调用组件。

## 服务拆分及远程调用
服务拆分注意事项：
1. 不同微服务，不重复开发相同业务。
2. 微服务数据独立，不要访问其他微服务的数据库。
3. 微服务可以将自己的业务暴露为接口，供其他微服务调用。

服务提供者与消费者：
- 服务提供者：一次业务中，被其他微服务调用的服务。（提供接口给其他微服务）
- 服务消费者：一次业务中，调用其他微服务的服务。（调用其他微服务提供的接口）
- 提供者与消费者角色是相对的。
- 一个微服务既可以是提供者又可以是消费者。（服务A调用服务B，服务B调用服务C，那么服务B）

## 远程调用的问题
1. 服务消费者该如何获取服务提供者的地址信息？（网页地址可能会随着环境变化，所以不建议使用）
2. 如果有多个服务提供者，消费者该如何选择？（服务提供者为一个集群）
3. 消费者如何得知服务提供者的健康状态？

## Feign
服务消费者之间的调用使用ribbon和feign（伪RPC）。feign默认已经集成了ribbon，feign主要是通过ribbon做负载均衡。
调用方式：RPC。远程过程方法调用，就是在本机调用服务器的方法（类似基于注解的WebService）
- 支持同步、异步调用。
- 客户端和服务器之间建立TCP连接，可以一次建立一个，也可以多个调用复用一次链接。
- PRC数据包小，protobuf，thrift
- RPC：编解码，序列化，链接，丢包，协议。

调用方式：Rest（HttpClient）。Http请求，支持多种协议和功能。
- 开发方便成本低。
- http数据包大。
- Java开发：HttpClient，URLConnection。
- 
## 搭建Feign
1. 导入依赖。
```xml
<!-- feign相关依赖,通过该模块的功能实现服务间的调用 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 服务消费者添加一个api包，在api包下新增一个接口充当被调用服务的客户端。
```java
package com.ly.api;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Map;

/**
 * @author ly
 * @Date: 2021/10/21 9:39
 */
//声明这个接口代表的是stocks-server的客户端
@FeignClient("stocks-server")
public interface StocksAPI {

    //1. 方法的签名必须和对应服务的接口一致
    //2. 在方法上指定对应接口的请求全路径
    @RequestMapping("/stocks/queryStocksNumberByGoodsId")
    //3. 所有参数必须使用@RequestParam注解，尤其是多个参数的方法
    public Map<String,Object> queryStocksNumberByGoodsId(@RequestParam("goodsId") Integer goodsId);

    @RequestMapping("/stocks/subStocksNumber")
    public Map<String,Object> subStocksNumber(@RequestParam("goodsId") Integer goodsId);
    
    //4. 如果参数是一个对象数据类型,建议使用Map数据类型,如果已经使用了对象数据类型,只能将该pojo类复制到本模板,也可以将所有pojo类放到一个模块并使所有模块继承
    @RequestMapping("/points/addPoints")
    //5. @RequestBody绑定对象类型参数
    public Map<String,Object> addPoints(@RequestBody Points points);
}

```

3. 在服务消费者的启动类上添加注解@EnableFeignClients("com.ly.api")指定api接口所有位置，将自动使用代理的方式生成接口的实现类并放入IOC容器中。
4. 被调用的服务，控制层方法的参数必须使用@RequestBody或@RequestParam注解与传入参数进行绑定，方法必须返回json格式数据。
```java
package com.ly.controller;

import com.ly.service.StocksService;
import com.netflix.discovery.converters.Auto;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

/**
 * @author ly
 * @Date: 2021/10/19 10:31
 */
@Controller
@RequestMapping("/stocks")
public class StocksController {
    /*
    * 1. 服务提供方所有的接口必须返回json格式数据
    * 2. 服务提供方的接口方法的参数必须使用@RequestParam()绑定
    * */

    @Autowired
    private StocksService stocksService;

    @RequestMapping("/queryStocksNumberByGoodsId")
    @ResponseBody
    public Map<String,Object> queryStocksNumberByGoodsId(@RequestParam("goodsId") int goodsId){
        int number = stocksService.queryStocksNumberByGoodsId(goodsId);
        Map<String,Object> result = new HashMap<>();
        result.put("number",number);
        //返回执行该方法的http状态码
        result.put("code",200);
        result.put("message","ok");
        System.out.println("查询库存方法被调用");
        return result;
    }

    @RequestMapping("/subStocksNumber")
    @ResponseBody
    public Map<String,Object> subStocksNumber(@RequestParam("goodsId") int goodsId){
        int number = stocksService.subStocksNumber(goodsId);
        Map<String,Object> result = new HashMap<>();
        if (number >= 0){
            result.put("code",200);
            result.put("message","ok");
        }else{
            result.put("code",500);
            result.put("message","error");
        }
        System.out.println("库存减一方法被调用");
        return result;
    }
}
```

5. 使用feign使用ribbon实现服务间调用的负载均衡。
```yaml
point-service:  #调用order-service的时候使用随机
		  ribbon:
		    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
        
        RoundRobinRule 轮询(默认)
        RandomRule 随机
        AvailabilityFilteringRule 会先过滤掉由于多次访问故障而处于断路器跳闸状态	的服务，还有并发的连接数超过阈值的服务，然后对剩余的服务列表进行	轮询
        WeightedResponseTimeRule 权重 根据平均响应时间计算所有服务的权重，响	应时间越快服务权重越大被选中的概率越高。刚启动时，如果统计信息不	足，则使用轮询策略，等信息足够，切换到 WeightedResponseTimeRule
        RetryRule 重试 先按照轮询策略获取服务，如果获取失败则在指定时间内重	试，获取可用服务
        BestAvailableRule 选过滤掉多次访问故障而处于断路器跳闸状态的服务，然后	选择一个ZoneAvoidanceRule 符合判断server所在区域的性能和server的		可用性选择服务
        策略选择：
        1、如果每个机器配置一样，则建议不修改策略 (推荐)
        2、如果部分机器配置强，则可以改为 WeightedResponseTimeRule

```

6. 服务间调用的超时解决（默认为1秒）。
```yaml
#防止ribbon超时
ribbon:
	# 改为4秒
  ReadTimeout: 4000 
  ConnectTimeout: 4000
```

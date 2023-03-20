# 7·Ribbon

- [7·Ribbon](#7ribbon)
  - [Ribbon是什么](#ribbon是什么)
  - [负载均衡的流程](#负载均衡的流程)
  - [使用步骤](#使用步骤)
  - [懒加载](#懒加载)

## Ribbon是什么
微服务间实现服务器负载均衡的组件。

## 负载均衡的流程
![图16-负载均衡工作流程](/docs/images/图16-负载均衡工作流程.jpeg)

## 使用步骤
1. 在启动类上添加注解@LoadBalanced
2. 通过定义IRule实现修改负载均衡规则。
```java
// 代码方式：在启动类中，定义一个新的IRule。作用范围是全局的。
@Bean
public IRule randomRule(){
	return new RandomRule();
}
```

```yaml
# 配置文件方式：添加新的配置修改规则。
userservice:
	ribbon:
  	NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则
```

## 懒加载
Ribbon默认采用懒加载，第一次访问时才会去创建LoadBalanceClient，请求时间很长。
饥饿加载则会在项目启动时创建，降低第一次访问的耗时。
```yaml
ribbon:
  eager-load:
    enabled: true # 开启懒加载
    clients: userservice # 指定对userservice服务饥饿加载
```

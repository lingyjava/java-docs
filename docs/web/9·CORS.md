# 9·CORS

- [9·CORS](#9cors)
  - [CORS是什么](#cors是什么)
  - [Spring Boot解决跨域](#spring-boot解决跨域)

## CORS是什么
CORS：Cross Origin Resource Sharing，跨域资源共享。
浏览器同源策略：限制不同源的资源共享。
同源：协议、域名、端口都相同。

## Spring Boot解决跨域

1. 在目标方法（接口）或类上添加`@CrossOrigin`注解。
```java
@RestController
public class Controller{
    @CrossOrigin
    @GetMapping("/list")
    public List<Emp> queryList(){}
}
```

2. 添加过滤器。
```java
@Configuration
public class CorsConfig{
    @Bean
    public CorsFilter cordFilter(){
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        // 添加允许请求的域
        corsConfiguration.addAllowedOrigin("*");
        // 添加允许请求的头信息
        corsConfiguration.addAllowedHeader("*");
        // 添加允许请求的方法类型
        corsConfiguration.addAllowedMethod("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**",corsConfiguration);
        return new CorsFilter(source);
    }
}
```

3. 实现WebMvcConfigurer接口，重写addCorsMappings方法
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
                // 允许访问的映射路径
        registry.addMapping("/**")
                // 允许访问的域
                .allowedOrigins("*")
                // 允许访问的请求类型
                .allowedMethods("GET","POST","PUT","DELETE")
                // 允许携带Cookie
                .allowCredentials(true)
                // 有效期3600秒，有效期类浏览器不必再询问
                .maxAge(3600)
                // 允许访问的头信息
                .allowedHeaders("*");
    }
}

```

charpter18 gateway下篇

18.1 服务发现路由规则
18.1.1 概述
18.1.2 案例
- 父工程
- consumer模块
  - 8000
  - 使用Feign远程调用
```java
@RequestMapping("/hello")
@RequestController
public class HelloController(){
  @Autowired
  HelloFeignService helloRemote;

  @GetMapping("/{name}")
  public String index(@PathVariable("name") String name) {
    return helloRemote.hello(name) + "\n" + new Date().toString();
  }
}
```
- eureka模块
  - 8761
- gateway模块
  - 9000
  - 依赖
    - spring-cloud-starter-gateway
    - spring-cloud-starter-netflix-eureka-client
  - application.yml
  ```yaml
  spring:
    application:
      name: sc-gateway-server
    cloud:
      gateway:
        discovery:
          locator:
            enabled: true # 开启基于法务发现的路由规则
            lowerCaseServiceId: true # 服务名使用小写，默认是大写
  server:
    port: 9000 #网关服务监听9000端口
  eureka:
    client:
      service-url:  #指定注册中心的地址
        defaultZone: http://localhost:8761/eureka
  logging:
    level:
      org.springframework.cloud.gateway: debug      

  ```

  - 主程序入口
  ```java
  @SpringBootApplication
  public class GatewayServerApplication {
    public static void main(String[] args) {
      SpringApplication.run(GatewayServerApplication.class, args);
    }
  }
  ```

- provider模块
  - 8001
```java
@ResController
public class HelloController {
  @GetMapping("/hello")
  public String hello(@RequestParam String name) {
    return "Hello, " + name + "!";
  }
}
```
18.2 Gateway Filter和Global Filter
18.2.1 概述
- Gateway Filter
  - 应用到单个路由，或者一个分组的路由上
  - 继承了ShortcutConfigurable
- Global Filter
  - 作用于所有路由
  - 没有任何继承
18.2.2 Gateway Filter 案例
- 计算路由耗时
18.2.3 Global Filter 案例
- 校验url权限

18.3 spring cloud gateway 实战
18.3.1 权重路由
- 权重路由断言工厂：WeightRoutePredicateFactory
- 1. 权重路由的使用场景
  - 灰度发布：95%的流量走服务V1版本, 5%流量走服务V2版本
- 2. 权重路由案例
  - 案例：gateway项目
  - 1 依赖：spring-cloud-starter-gateway
  - 2 入口程序：
  ```java
  @SpringBootApplication
  public class GatewayApplication() {
    public static void main(String[] args) {
      SpringApplication.run(GatewayApplication.class, args);
    }
  }
  ```
  - 3 application.yml配置权重路由信息
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service1_v1
            uri: http://localhost:8081/v1
            predicates:
              - Path=/test
              - Weight=service1, 95 # 路由权重信息  
        - id: service1_v2
            uri: http://localhost:8081/v2
            predicates:
              - Path=/test
              - Weight=service1, 95  # 路由权重信息  
```    
  - 4 provider工程
  ```java
  @RestController
  public class ServiceController {
    @GetMapping(value = "/v1")
    public String v1(){
      return "v1";
    }
    @GetMapping(value = "/v2")
    public String v2(){
      return "v2";
    }
  }

  ```

18.3.2 https的使用技巧
- https通常做法：通过Nginx来配置SSL证书
- gateway统管所有API出口和入口，就要支持https，将生成的https证书放到gateway应用的类路径下面。
- 案例
- 父工程
- eureka模块
  - 8761
- gateway-https模块
  - 8080
  - 带有https证书的server，使用https协议访问
  - 依赖：
    - spring-cloud-starter-gateway
    - spring-cloud-starter-netflix-eureka-client
  - application.yml配置https
    - https证书放在和resources目录下
  ```yaml
  server:
  ssl:
    key-alias: spring
    enabled: true
    key-password: spring
    key-store: classpath:selfsigned:jks
    key-store-type: JKS
    key-store-provider: SUN
    key-store-password: spring
  ````
- service-a模块
  - 8071
  ```java
  @RestController
  public class  TestController(){
    @GetMapping("/test")
    public String prefixpath(){
      return "https to http";
    }
  }
  ```
- service-b模块
  - 8072

- 3 测试
  - https://localhost:8080/service-provider/test
  - 报错：no an SSL/TLS record
  - gateway进来的是https协议
  - gateway将https请求转发调用http服务问题，
  - 一般企业内网没有必要加https

- 4 解决方案：都需要实现GlobalFilter接口
  - 1）在LoadBalanceClientFilter执行之前将Https修改为Http.
  - 2) 在LoadBalanceClientFilter执行之后将Https修改为Http.

18.3.3 集成swagger
- 案例
- 父工程
- eureka模块
  - 8761
- gateway模块
  - 8081
  - 带有https证书的网关server，使用https协议访问
  - 依赖：
  - spring-cloud-starter-gateway
  - spring-cloud-starter-netflix-hystrix
  - springfox-swagger-ui
  - springfox-swagger2
- 通过GatewaySwaggerProvider实现SwaggerResourcesProvider接口，获取SwaggerResource. (Swagger不支持webflux项目，不能再Gateway配置SwaggerConfig)
```java
public class GatewaySwaggerProvider implements SwaggerResourcesProvider {
  public static final String API_URI = "/v2/api-docs";
}
```
- service模块
  - 8055





















18.3.4 限流
18.3.5 动态路由

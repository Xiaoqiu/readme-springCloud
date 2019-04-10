# charpter 6 Hystrix实践运用
- 设计目标
- 基础用法
- 实战技巧

## 6.1 Hystrix 概述
### 6.1.1 解决什么问题
- 分布式应用容错
### 6.1.2 设计目标
- 1 通过客户端库对延迟和故障进行保护和控制
- 2 在一个复杂的分布式系统中停止级联故障
- 3 快速失败和迅速恢复
- 4 在合理的情况下回退和优雅地降级
- 5 开启近实时监控、告警和操作控制
- 底层大量使用了Rxjava
## 6.2 Hystrix 实践运用
### 6.2.1 入门示例
- 1 创建client-service工程
- 2 启动断路器模式
  - 在启动类添加@EnableHystrix
- 3 增加@HystrixCommand和降级fallback
```java
@HystrixCommand(fallbackMethod="defaultUser")
public String getUser(String username) throws Exception {
  if(username.equals("spring")) {
    return "this is real user";
  }else {
    throws new Exception();
  }
}

//出错则调用该方法返回预设友好错误
public String defaultUser(String username) {
  return "The user does not exist in this system";
}
```
- 效果展示
  - 调用接口，当用户名等于spring时返回正确的信息，当不是spring时则抛出异常，降级处理返回友好的提示。
  - 浏览器：http://localhost:8888/getUser?username=spring
### 6.2.2 Feign中使用断路器
- Feign默认自带Hystrix的功能，通过配置打开
- 1 创建consumer工程
  - 使用@FeignClient定义接口，配置降级回退类UserServiceFallback
```java
@FeignClient(name = "sc-provider-service", fallback = UserServiceFallback.class)
public interface IUserService {
  @RequestMapping(value = "/getUser", method = RequestMethod.GET)
  public String getUser(@RequestParam("username") String username);
}
```

- 2 创建降级fallback类

```java
@Component
public class UserServiceFallback implements IUserService {
  //出错则调用该方法返回友好错误
  public String getUser(String username) {
    return "The user does not exits in this system, please confirm username";
  }
}

```
- 3 结果展示
- 不启动hystrix
  - 不启动sc-provider-service工程
  - feign.hystrix.enable=false
  - 访问http://localhost:8888/getUser?username=hello
    - 返回500
- 启用hystrix
- 不启动sc-provider-service工程
- feign.hystrix.enable=true
- 访问http://localhost:8888/getUser?username=hello
  - 显示友好信息

### 6.2.3 Hystrix dashboard
- 1 搭建相关工程
  - 创建eureka工程
  - 创建hello-service工程 ：
  - 创建provider-service工程 ：提供一个接口返回信息


- 创建hello-service工程
    - 1. 依赖
    ```xml
    <dependency>
      <groupId> org.springframework.boot</groupId>
      <artifactId> spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
    - 2. 配置
    ```yml
    management:
      security:
        enable: false
      endpoints:
        web:
          exposure:
            include: hystrix.stream
    feign:
      hystrix:
        enabled: true        
    ```
  - 3 hello-service中定义一个ProviderService的FeignClient负责调用接口
```java
@FeignClient(name = "sc-provider-service")
public interface ProviderService {
  @RequestMapping(value = "/getDashboard",method = RequestMethod.GET)
  public List<String> getProviderData();
}
```

- 2 搭建Hystrix Dashboard工程
  - 增加Hystrix Dashboard依赖
```xml
<dependency>
  <groupId> org.springframework.cloud</groupId>
  <artifactId> spring-cloud-starter-netflix- hystrix-dashboard</artifactId>
</dependency>
```
  - 增加@EnableHystrixDashboard注解在启动类


- 3 Hystrix Dashboard运行结果和解释
  - 启动Hystrix Dashboard,hello-service两个工程
  -
### 6.2.4 Turbine聚合Hystrix

### 6.2.5 Hystrix异常机制和处理

### 6.2.6 配置说明
### 6.2.7 线程调整和计算
### 6.2.8 请求缓存
### 6.2.9 Request Collapser
### 6.2.10 线程传递并发策略
### 6.2.11 命令注解

## 6.3 本章小结

# charpter 5 Ribbon实践运用

## 5.1 Ribbon概述

### 5.1.1 Ribbon与负载均衡

- 软负载：Nginx
- 硬负载：F5
- 集中式负载均衡：Nginx ,F5,
  - 位于因特网与服务提供者之间，负载把网络请求转发到各个提供单位。
- 进程内负载均衡：Ribbon, IPC组件(Inter-Process-Communication)
  - 指从一个实例库选取一个实例进行流量导入。
- 客户端负载均衡
- Feign与Zuul中默认集成Ribbon
- 云原生12要素：

### 5.1.2 入门案例

- 集成Eureka : 从注册中心获取服务列表

- 1 创建Maven父级pom工程

- 2 创建Eurecka组件工程

- 3 创建源服务工程client-a

- 4 创建Ribbon客户端工程

- ```java
  @RestControoler
  public class TestController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/add")
    public String add(Integer a, Integer b){
      String result = restTemplate.getForObject("http://Client-A/add?a=" + a + "&b=" + b, String.class);
      System.out.println(result);
      return result;
    }
  }
  ```

- 5 测试

  - 1）启动Eureka Server之后，修改client-a的服务器端口号为7070与7075分别启动。访问http://localhost:8888
  - 2) 访问http://localhost:7777/add?a=100&b=300

## 5.2 Ribbon实践

### 5.2.1 Ribbon负载均衡策略与已定义配置

- 负责均衡策略
  - 1 随机策略：RandomRule
    - 随机选择server
  - 2 轮询：RoundRibbonRule
    - 按照循环选择server
  - 3 重试：RetryRule
    - 在一个配置时间段内当选择server不成功，则一直尝试选择一个可用的server
  - 4 最低并发策略 ： BestAvailableRule
    - 逐个考察server，如果server断路器打开，则忽略，选择其中一个并发链接最低的server
  - 5 可用过滤策略 ： AvailablilityFilterRule
    - 过滤掉一直失败并标记为circuir tripped的server，过滤掉那么高并发链接的server，(active connections 超过配置阀值)
  - 6 响应时间加权 ResponseTimeWeightedRule
    - 根据server的响应时间分配权重。响应时间越长，权重越低，被选择到的概率越小；响应时间越短，权重越高，被选择到的概率越高。这个策略很贴切，综合了各种因素，如果：网络，硬盘，IO等，这些因素直接影响着响应时间。
  - 7 区域权衡 : ZoneAvoidanceRule
    - 综合判断server所在区域的性能和server的可用性轮询选择server，并且判定一个AWS Zone的运行性能是否可用，提出不可用的Zone

- 1 全局策略设置： 添加一个配置类

```java
@Configuration
public class TestConfiguration {

  @Bean
  public IRule ribbonRule() {
    return new RandomRule();
  }
}
```
- 2 基于注解的策略设置
  - 针对某个源服务设置特有的策略。可以通过@RibbonClient注解。
  - 配置文件
  ```java
    @Configuration
    @AvoidScan // 空注解
    public class TestConfiguration {

      @Autowired // 针对客户端的配置管理器
      IClientConfig config;

      @Bean
      public IRule ribbonRule () {
        return new RandomRule();
      }
    }

  ```
```java
// 启动类
@RibbonClient(name = "client-a",configuration = TestConfiguration.class)
@ComponetScan(excludeFilters = {@ComponetScan.Filter(type = FilterType.ANNOTATION, value = {AvoidScan.class})})
```
  - @RibbonClient的指名了对client-a服务使用的策略是记过TestConfiguration配置的。
  - @ComponetScan注解上面的意思是，让spring不去扫描 @AvoidScan注解标记的注解类，因为我们的配置对单个源服务生效，所以不能应用于全局。
  - @RibbonClients注解：可以对多个源服务进行策略制定。
```java
@RibbonClients( value = {
    @RibbonClient(name = "client-a", configuration = "TestConfiguration.class"),
    @RibbonClient(name = "client-b", configuration = "TestConfiguration.class")
})
```

- 3 基于配置文件的策略设置 (推荐使用)
  - 语法：<client name>.ribbon.*
```yml
client-a:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 5.2.2 Ribbon超时与重试
- 配置超时和重试策略:
```yml
client-a:
  ribbon:
    ConnectTimeout: 30000
    ReadTimeout: 30000
    MaxAutoRetries: 1 # 对第一次请求的服务的重试次数
    MaxAutoRetriesNextServer: 1 # 要重试的下一个服务的最大数量（不包括第一个服务）
    OkToRetryOnAllOperations: true
```

### 5.2.3 Ribbon的饥饿加载
- Ribbon在进行客户端负载均衡的时候，并不是在启动时就去加载上下文，而是在实际请求的时候才去创建。
- 配置开启饥饿加载：即在启动的时候就去加载应用上下文。
```yml
ribbon:
  eager-load:
    enabled: true
    clients: client-a, client-b, client-c
```

### 5.2.4 利用配置文件自定义Ribbon客户端
- 使用配置文件来定制Ribbon客户端。指定默认的加载类。
- 指定ILoadBalancer的实现类
- 指定IRule的实现类
- 指定IPing实现类
- 指定ServerList实现类
- 指定ServerListFilter实现类

### 5.2.5 Ribbon脱离Eureka的使用
- 在Ribbon客户端自定指定源服务地址。
```yml
## 在Ribbon中禁用Eureka的功能
ribbon:
  eureka:
    enable: false

## 对源服务设定地址列表
client:
  ribbon:
    listOfServers: http://localhost:7070, http://localhost:7071
```

## 5.3 Ribbon进阶

### 5.3.1 核心工作原理
- 核心接口
  - IClientConfig
    - 描述：定义管理配置的接口
    - 默认实现类：DefaultClientConfigImpl
  - IRule
    - 负载均衡策略接口
    - ZoneAvoidanceRule
  - IPing
    - 定期ping服务检查可用性的接口
    - DummyPing
  - ServerList<Server>
    - 获取服务列表方法是的接口
    - ConfigurationBaseServerList
  - ServerListFilter<Server>
    - 定义特定期望获取服务列表方法的接口
    - ZonePreferenceServerListFilter
  - ILoadBalancer
    - 定义负载均衡选择服务的核心方法的接口
    - ZoneAwareLoadBalancer
  - ServerListUpdater
    - 为DynamicServerListLoadBalancer定义动态更新服
    务列表的接口
    - PollingServerListUpdater
- 从启动到负载均衡选择服务实例的过程: RestTemplate达到负载均衡
  - 基本使用方式需要注入一个RestTemplate的Bean,并且使用@LoadBalanced注解，具备负载均衡能力。
  -
### 5.3.2 负载均衡策略源码发导读



## 5.4 本章小结

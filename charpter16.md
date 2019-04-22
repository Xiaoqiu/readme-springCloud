charpter16 链路监控
- sleuth例子
  - 父pom
    - 依赖：spring-cloud-starter-sleuth
    - spring-cloud-starter-openfeign
    - spring-boot-starter-web
  - sleuth-consumer: 8081
  - sleuth-provider: 8082
  - 使用Feign RestTemplate

## 16.3 sleuth深入用法
### 16.3.1 TraceFilter
- 对于HTTP的接口服务，接收客户端span信息的方式是filter
- 对span信息自定义的修改，注册一个自己的filter

### 16.3.2 Baggage
- 存储在span上下文中的一组key/value键值对。
- 让数据跟着sleuth一起往后传递，例如登录信息的传递

### 16.3.3 案例
- maven父级pom工程
  - spring-cloud-starter-sleuth
  - spring-cloud-starter-openfeign
  - spring-boot-starter-web

- sleuth-consumer模块:
  - 使用自定义的filter: 获取前端传来的sessionId,放入Baggage
  - 使用feign调用provider:把sessionId传给provider
  - sessionFilter: 将自定义信息放入span中。
  ```java
  /**
    自定义过滤器
    获取当前的sessionId, 放入baggage中
    不是所有的请求都需要往后传递，所以会对一些请求跳过执行
  **/
  @Component
  @Order(TraceWebServletAutoConfiguration.TRACING_FILTER_ORDER + 1)
  public class SessionFilter extends GenericFilterBean {
    private Pattern skipPattern = Pattern.compile(SleuthWebProperties.DEFAULT_SKIP_PATTERN);

    public void doFilter(ServletRequest request,ServletResponse response, FilterChain filterChain) throw IOException, ServletException {
      if (!(request instanceof HttpServletRequest) | !(response instanceof HttpServletResponse)) {
          throw new ServletException("filter just support http requests");
      }

      HttpServletRequest httpRequest = (HttpServletRequest)request;

      boolean skip = skipPattern.matcher(httpRequest.getRequestURI()).matches();

      if(!skip) {
        // 将sessionId放入到baggage中
        ExtraFieldPropagation.set("SessionId", httpRequest.getSession().getId());
      }

      filterChain.doFilter(request, response);
    }

  }
  ```
  - 创建配置文件
    - bootstrap.yml
    ```yaml
    server:
      port: 8081
    spring:
      application:
        name: sleth-consumer
      sleuth:
        baggage-keys: #注意，sleuth2.0.0之后，baggage的key必须在这里配置才能生效
          - SessionId    
    ```
- sleuth-provider模块 ： 8082
```java

/**
sleuth-provoder 对外服务接口
**/
@RestController
public class ProviderController {
  @GetMapping("/sayHello")
  public String hello(String name){
    return "hello," + name + ", sessionId is " + ExtraFieldPropagation.get("SessionId");
  }
}

```

- 4) 测试
  - 启动sleuth-consumer：8081和sleuth-provider：8082
  - url: http://localhost:8081/hello?name=scbook

16.4 skyWalking （非侵入性APM）
- 分布式追踪和上下文传输
- 应用，实例，服务性能指标分析
- 根源分析
- 应用拓扑分析
- 应用和服务依赖分析
- 慢服务检测
- 性能优化
16.4.3 skyWalking主要特性
- 多语言探针或类库
- Java自动探针，追踪和监控程序时，不需要修改源码
- 社区提供的语言探针：.NET Core Nodejs
- 多种后端存储：ElasticSearch. h2
- 支持OpenTrancing: Java自动探针和OpenTracing API协同工作
- 轻量级，完善的后端聚合和分析功能
- 现代化Web UI
- 日志集成
- 应用，实例，和服务的告警
- 支持接收其他跟踪器数据格式
- ZipkinJSON, Thrift, Protobuf v1 和v2格式，由OpenZipkin库提供支持。
- Jaeger采用Zipkin Thrift或JSON V1/V2格式

16.4.4 skyWalking整天架构
- collector: 对收集到的数据进行分析和聚合，最后存储到ElasticSearch或H2中，一般H2用于测试。
- agent: 可以使用SDK的形式接入，也可以使用非入侵的agent形式接入，agent将数据转化为skyWalking Trace数据协议，通过http或者gRPC发送到collector
- web
- storage

16.5 skyWalking实践
- zuul网关
- service-a: 通过feign远程调用service-b
- service-b
- eureka注册中心

16.5.1 父工程创建
- 添加：web,测试，端点的依赖
- spring-boot-starter-web
- spring-boot-starter-test
- spring-boot-starter-actuator

16.5.2 创建eurecka-server-skywalking工程： 服务注册中心
- 1）创建工程模块
  - ch16-3-eureka-server-skywalking
- 2）配置依赖
  - spring-cloud-starter-netflix-eureka-server
- 3）编写主程序入口代码
  ```Java
  @SpringBootAppliation
  @EnableEurekaServer
  public class EurekaServerApplication {
    public static void main(String[] args){
      SpringApplication.run(EurekaServerApplication.class)
    }
  }
  ```
- 4) 配置文件
  - application.yml
  ```yaml
  server:
    port: 8761
  eureka:
    instance:
      hostname: localhost
    client:
      registerWithEureka: false
      fetchRegistry: false
      serviceUrl:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/  
    server:
      waitTimeInMsWhenSyncEmpty: 0
      enableSelfPreservation: false

  ```
- 服务注册中心编写完成

16.5.3 创建zuul-skywalking: 网关
- 1) 创建工程模块


16.5.4
16.5.5
16.5.6 skyWalking collector 基础环境安装
- 配置es, elasticSearch.yml
- 启动es
- 下载skywalking目录
  - agent
  - bin
  - startup
  - config
  - log
  - webapp
  - 端口：8080， 10800，11800，1280
  - 修改端口：config/application.yml
  - 修改端口：webapp/webapp.xml
  - elasticSearch.yml的cluster.name和config.application.yml的对应的clusterName相同

- 启动collector和web
  - bin目录： ./startup.sh
  - 访问地址：localhost:8080
  - 默认用户名/密码：admin/admin
  - web（UI进程）通过127.0.0.1:10800访问本地collector

16.5.7 使用agent启动服务和监控查看
- service-a
- service-b
- service-eureka
- service-zuul
- 主要放被监控的jar和agent,每个应用使用一个对应的agent进行启动
- 把下载的skyWalking中的agent目录拷贝出来，运行服务的时候使用。
- 修改agent/config/agent.conf文件
  - application_code=service-a
  - application_code=service-b
  - application_code=service-service-zuul
  - application_code=service-service-eureka
- 启动监控服务
```bash
  java -javaagent:skywalking

```



















16.6 pinpoint (非侵入性APM）

16.7 pinpoint实践

charpter 8

8.1 工作原理

- zuul filter的主要特性：
  - filter 类型
  - filter的执行顺序
  - filter的执行条件
  - filter的执行效果

- zuul提供了一个动态读取，编译，运行这些filter的机制。
- filter直接不直接通向，请求线程通过RequestContext共享状态，内部通过ThreadLocal来手机自己需要的状态数据。
- 不同Filter的执行逻辑核心：com.netflix.zuul.http.ZuulServlet类中定义。

-  PostFilter的抛错分两种：流程终点不只是post filter, 还有可能是error filter

  - 在post Filter抛错之前，pre、route filter没有抛错，此时进入error filter，进入zuulException逻辑，打印堆栈信息。然后返回status=500的error信息。
  - 在post filter抛错之前，pre , route filter已经抛错。此时不会打印堆栈，直接返回status=500的error信息。

- zuul 一共有四种不同生命周期的filter：

  - pre: zuul按照规则路由到下级服务之前执行。鉴权，

  - route：这些filter是zuul路由的动作执行者，是httpClient, okhttp或ribbon构建发送原始http请求的地方。

  - post：源服务返回结果或者异常信息发生后执行，对返回信息做处理，可以在这个filter进行。

  - error: 整个生命周期内如果发送异常，会进入error filter，可做全局的异常处理。

  - >
    >
    >在 Filter 之间， 通过 com. netflix. zuul. context. RequestContext 类 来 进行 通信， 内部 采用 ThreadLocal 保存 每个 请求 的 一些 信息， 包括 请求 路 由、 错误 信息、 HttpServletRequest、 HttpServletResponse， 这使 得 一些 操作 是 十分 可靠 的， 它 还 扩展 了 ConcurrentHashMap， 目的 是 为了 在 处理 过程中 保存 任何 形式 的 信息， 后面 会对 它 进行 具体 讲解。
    >



8.1.2 zuul 原生filter

- Zuul server 如果使用@Enable-ZuulProxy, 搭配Spring boot actuator, 多两个管控端点。
  - 1）/routes:
  - 2) /filters
  - 3) /zuul.<SimpleClassName>.<filterType>.disable=true:

8.1.3 多级业务处理

- 1 实现自定义filter

  - 实现ZuulFilter类，一个抽象类：

    - String filterType()

    - int filterOrder()

    - boolean shouldFilter()

    - Object run()

    - ```java
      public class FirstPreFilter extends ZuulFilter {
        @Override public String filterType() {
          return PRE_ TYPE;
        }
        @Override public int filterOrder() {
          return 0;
        }
        @Override public boolean shouldFilter() {
          return true;
        }
        @Override public Object run() throws ZuulException {
          System. out. println(" 这是 第一个 自定义 Zuul Filter！");
          return null;
        }
      }

      // 静态导入FilterConstants类里定义了常量
      // 启动类注入bean容器
      @Bean
      public FirstPreFilter firstPreFilter(){
        return new FirstPreFilter();
      }

      ```

    -

- 2 业务处理实践

-



8.2 zuul 权限集成

8.2.1 应用权限概述

- 1 自定义权限认证filter
- 2 OAuth2.0 + jwt

8.2.2 Zuul + OAuth2.0 + JWT实践

- 1 zuul-server编写说明
- 2 auth-server编写说明

8.3 zuul限流

8.3.1

-  流量排队，限流，分流
- 限流算法 ： 漏铜，令牌桶

8.3.2 限流实战

- 粒度策略

- 粒度临时变量存储方式

- 1 zuul-server编写说明

  - 依赖：spring-cloud-zuul-ratelimit

  -  编写bootstrap.yml配置文件： 采用本地存储，3秒的时间窗口内不能有超过2次的接口调用。

  - ```yaml
    server:
    	port: 5555
    eureka:
    	client:
    		serviceUrl:
    			defaultZone: http://127.0.0.1:8888/eureka/
    zuul:
    	routes:
    		client-a:
    			path: /client/**
    			serviceId: client-a
    	ratelimit:
      	key-prefix: sprngcloud-book # 按粒度拆分的临时变量key前缀
      	enabled: true # 启动开关
      	repository: IN MEMORY # key存储类型，默认是IN_MEMORY本地存储，此外还有其他形式
      	bebind-proxy: true # 表示代理之后
      	default-policy:  # 全局限流策略，可单独细化服务粒度
      		limit: 2 # 在一个单位时间窗口的请求数量2次
      		quota: 1 # 在一个单位时间窗口的请求的时间限制
      		refresh-interval: 3 # 单位时间窗口为3秒
      		type:
      			- user # 可指定用户粒度
      			- origin # 可指定客户端地址粒度
      			- url # 可指定url粒度

    ```

- 2 测试
  - ch8-2-eureka-server,
  - ch8-2-zuull-server,
  - ch8-2-client-a
  -

8.4 zuul 动态路由

-

8.5 zuul灰度发布

8.6 zuul文件上传
8.6.1 文件上传实践
- zuul-server编写说明
```yaml
spring:
  application:
    name: zuul-server
  servlet: #spring boot2.0之前是http
    multipart:
      enabled: true # 使用http multipart上传处理
      max-file-size: 100MB #设置单个文件的最大长度默认1M,如不限制配置为-1
      max-request-size: 100MB # 设置最大的请求文件大小，默认10M, 如不限制配置为-1
      file-size-threshold: 1MB # 当上传的文件达到MB时候进行磁盘写入
      location: / # 上传文件的临时目录
      hystrix:
        command:
          default:
            execution:
              isolation:
                thread:
                  timeoutInMilliseconds: 30000  # 默认的超时时间为1秒，如果上传大文件，为避免超时，稍微设置大一点。
      ribbon:
        ConnectTimeout: 3000
        ReadTimeout: 30000

```
  - 接收前端传过来的文件的接口。
```java
@Controller
public classs ZuulUploadController {
  @PostMapping("/upload")
  @ResponseBody
  public String uploadFile(@RequestParam(value = "file", required = true)MultipartFile file) throw IOException {
    byte[] bytes = file.getBytes();
    File fileToSave = new File(file.getOriginalFilename());
    FileCopyUtils.copy(bytes,fileToSave);
    // 返回上传文件的绝对路径
    return fileToSave.getAbsolutePath();
  }
}
```
- 2 测试
  - 启动eureka-server
  - zuul-server

8.6.2 文件上传乱码解决
- 路径改为：http://localhost:5555/zuul/upload
- 改为使用zuul servlet 上传文件。  
8.7 zuul 实用技巧
8.7.1 饥饿加载
- zuul使用ribbon来调用远程服务，
- 第一次经过zuul的调用往往会去注册中心读取服务注册表，初始化ribbon负载信息，这是一种赖加载策略
- 可以设置为启动zuul的时候饥饿加载应用程序上下文。
```yaml
# 开启饥饿加载
zuul:
  ribbon:
    eager-load:
      enabled: true
```
8.7.2 请求体修改
- 新增一个PRE类型额filter对请求体进行修改
```java
@Configuration
public class ModifyRequestEntityFilter extends ZuulFilter {
  @Override
  public String filterType(){
    return PRE_TYPE;
  }
  @Override
  public int filterOrder() {
    return PRE_DECORATION_FILTER_ORDER + 1;// 6
  }
  @Override
  public boolean shouldFilter() {
    return true;
  }
  @Override
  public Object run() throws ZuulException{
    try {
      RequestContext context = RequestContext.getCurrentContext();
      String charset = context.getRequest().getCharacterEncoding();
      InputStream in = (InputStream) context.get("requestEntity");
      if (in == null) {
        in = context.getRequest().getInputStream();
      }
      //将输入流转为字符串，再做添加参数处理
      String body = StreamUtils.copyToString(in,Charset.forName(charset));
      //新增参数
      body += "&weigiht=140";
      byte[] bytes = body.getBytes(charset);
      context.setRequest(new HttpServletRequest(context.getRequest()){
          @Override
          public ServletInputStream getInputStream() throws IOException {
            return new ServletInputStreamWrapper(bytes);
          }
          @Override
          public int getContentLength() {
            return bytes.length;
          }
          @Override
          public long getContextLengthLong(){
            return bytes.length;
          }
        });
      } catch (IOException e) {
        ReflectionUtils.rethrowRuntimeException(e);
      }
      return null;
  }
}
```
- ModifyRequestEntityFilter的次序为PRE类型Filter最后一级
- shouldFilter()方法可以设定filter执行的特定条件

8.7.3 使用okhttp替换HttpClient
- okHttp优点：
  - 可以合并多个对于同一个HOST的请求
  - 使用链接复用机制
  - 使用GZIP压缩
  - 对响应缓存
- zuul 使用okhttp
  - 依赖：okhttp的maven依赖
  - 配置文件：
    ```yaml
    # 禁用httpclient并开启OKhttp
    ribbon:
      httpclient:
        enabled:false
      okhttp:
        enabled:true  
    ```
8.7.4 重试机制

- 重试配置spring retry, ribbon使用
- 依赖：spring-retry
- 配置：
```yaml
zuul:
  retryable: true # 开启重试
  ribbon:
    ConnectTimeout: 3000
    ReadTimeout: 60000
    MaxAutoRetries: 1 # 对第一次请求的服务的重试次数
    MaxAutoRetriesNextServer: 1 # 要重试的下一个服务的最大数量（不包含第一个服务）
    OkToRetryOnAllOperations: true
 spring:
   cloud:
     loadbalancer:
       retry:
         enabled: true # 内部默认已经开启，这里列出来说明这个参数比较重要。
```
- 也可以对单个映射规则进行重试：zuul.routes.<route>.retryable=true

8.7.5 Header传递
- zuul一般对请求的head进行处理，不影响原来请求。有一个重要的类RequestContext
- zuul-server:
```java
  public class HeaderDeliverFilter extends ZuulFilter {
    //....
    @Override
    public Object run() throws ZuulException {
      RequestContext context = RequestContext.getCurrentContext();
      //动态增加一个header传递到下游服务，十分实用，这种方式与敏感头相反。注意信息安全性
      context.addZuulRequestHeader("result","to next service");
      return null;
    }
  }
```

8.7.6 整合swagger2实现对源服务的依赖
```java
@Configuration
@EnableSwagger2
public class swaggerCongfig {
  @Autowired
  ZuulProperties properties;

  public SwaggerResourcesProvider swaggerResourcesProvider() {
    return () -> {
      List<SwaggerResource> resources = new ArrayList<>();
      properties.getRoutes().values().stream().forEach(route -> resources.add(createResource(route.getServiceId(),route.getServiceId(),"2.0")));
      return resources;
    };
  }

  private SwaggerResource createResource(String name, String location, String version){
    SwaggerResource swaggerResource = new SwaggerResource();
    swaggerResource.setName(name);
    swaggerResource.setLocation("/" + location + "/v2/api-docs");
    swaggerResource.setSwaggerVersion(version);
    return swaggerResource;
  }
}
```



















8.8 文章小结

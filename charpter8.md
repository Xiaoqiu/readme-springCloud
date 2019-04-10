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

8.4 zuul 动态路由

8.5 zuul灰度发布

8.6 zuul文件上传

8.7 zuul 实用技巧

8.8 文章小结






















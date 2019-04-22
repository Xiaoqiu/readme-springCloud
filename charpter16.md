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
- comsumer
  - 使用自定义的filter，获取前端传来的sessionId,放入baggage中，
  - 通过feign调用的方式，将sessionId传递给provider
- provider
-

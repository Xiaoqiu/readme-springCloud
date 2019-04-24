charpter19 spring cloud gRpc上篇
- Feign内部通讯，相对于dubbo， gRpc等RPC框架性能十分低。
-
19.1 为什么需要gRPC
- 微服务一般外部使用http
- 内部使用Feign, 底层使用HTTP协议进行服务调用
- gRPC的吞吐量是HTTP/JSON的客户端的3倍原因：
  - 1 gRPC采用了Proto Buffer作为序列化工具
  - 2 gRPC采用HTTP2协议，头部压缩，对连接进行复用，减少TCP连接次数。
  - 3 gRPC采用Netty作为IO处理框架，提供性能。
19.2 gRPC简介
  - 客户端可以通过RPC远程调用方法
19.3 核心概念
19.3.1 服务定义
- gRPC使用protocol buffers作为接口定义语言
```java
service HelloService {
  rpc SayHello (HelloRequest) returns (response)
}

message HelloRequest {
  string greeting = 1;
}
message HelloResponse {
  string reply = 1;
}
```
- gRPC 可以定义4中服务方法：
  - 1）Unary RPC (一元RPC):
  - 2) 服务端流式RPC:
  - 3) 客户端流式RPC：
  - 4）双向流式RPC:

19.3.2 使用API

19.3.3 同步vs异步


19.4 生命周期

19.5 依赖于protocol buffers
19.5.2 使用protocol buffers的maven插件
- 依赖：
  - protobuf-java
  - pom文件需要的配置项：
  - proto的源IDL文件地址：默认：${project.basedir}/src/main/proto (.proto文件)
  - 编译后输出的文件地址：默认：${project.build.directory}/generated-sources/protobuf/java
  - protocol buffers的编译地址：

19.6 基于http2
- 1 二进制流
  - http2是一个二进制协议，每个帧属于一个特定的流。一个响应或请求可以由多个帧组成。
- 2 多路复用
  - 客户端和服务端可以包含多个并发流，可以同时发生消息（请求或响应）。流的多路复用，提高了连接的利用率。
- 3 头压缩
  - 头部包含cookie，http是无状态的，每次都包含cookie，让服务器记住客户端的状态。
- 4 服务器推送
  - 客户端请求html，服务端吧所需要的css文件都推送给客户端。客户端需要主动开启，可以随时选择终止。
- gRPC多用于移动端和服务端的通讯
- gRPC优点：
  - 双向流
  - 流控
  - 头压缩
  - 单TCP连接上的多复用请求


19.7 基于netty进行IO处理
- 
19.8 案例
19.9 本章小结

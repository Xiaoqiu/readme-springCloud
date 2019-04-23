charpter 17 gateway上篇
- 第二代网关： spring cloud gateway

17.1 概述
17.1.1
- 替换zuul
- 基于filter链的方式提供：安全，监控/埋点，限流
- 基于技术
  - spring 5.0
  - spring boot 2.0
  - project reactor


17.1.2 核心概念
- 路由：网关最基础的部分。一个ID,一个目的url，一组断言工厂和一组filter组成。

17.4 gateway路由断言
17.4.1 after路由断言工厂
17.4.2 before路由断言工厂
17.4.3 between路由断言工厂
17.4.4 cookie路由断言工厂
17.4.5 header路由断言工厂
- 1）
17.4.6 host路由断言工厂
17.4.7 method路由断言工厂
17.4.8 query路由断言工厂
17.4.9 remoteAddr路由断言工厂

17.5 gateway的内置filter
- 对请求和响应修改
- 分为7类：
  - header
  - parameter
  - path
  - redirect
  - hytrix
  - ratelimiter


17.5.1 AddRequestHeader过滤工厂

17.5.2 AddRequestParameter过滤工厂
17.5.3 RewritePath过滤器
17.5.4 AddReponseHeader过滤器
17.5.5 StripPrefix过滤器
17.5.6 Retry过滤器
17.5.7 Hytrix过滤器

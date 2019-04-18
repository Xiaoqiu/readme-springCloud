# charpter 12 spring cloud config下篇

## 12.1 服务端git配置详解与实践
12.1.1 git的多种配置详解概述
- config server中关于git的配置
- git的uri占位符
- 模式匹配和多个存储库
- 路径搜索占位符

12.1.2 git的uri占位符
- 支持：{application} {profile} {label}
```yaml
## 服务端应用配置application.yml
spring:
  cloud:
    config:
      server:
        git:
          uri: htpps://gitee.com/zhongzunfa/{application}
          serarch-paths: SC-BOOK-CONFIG
  application:
    name: ch12-1-config-server-placeholders
server:
  port: 9090      
loggint:
  level:
    root: debug           
```
12.1.4 路径搜占位符
- 不同的项目对不同的配置文件进行路径配置，
```yaml
searchPaths: "{application}"
或
searchPaths: *{application}*
```

## 12.2 关系型数据库配置的配置中心的实现
### 12.2.1 config基于mysql的配置概述

























## 12.3 非关系型数据库的配置中心的实现
## 12.4 config实用技能
## 12.5 config功能扩展
## 12.6 高可用部分
## 12.7 spring cloud 与appolo配置使用
## 12.8 spring cloud 与appolo结合使用实践
## 12.9 本章小结

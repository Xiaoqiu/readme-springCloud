# charpter 11 spring cloud config上篇
## 11.1 spring cloud config 配置中心概述
11.1.1 什么是配置中心
- 2
- 3 配置中心应具备的功能：
  - open API
  - 业务无关性
  - 配置生效监控：*
  - 一致性K-V存储
  - 统一配置实时推送
  - 配合灰度与更新
  - 配置全局恢复、备份与历史
  - 高可用集群
  - 整个支撑体系，两类：
    - 运维管理体系
      - 启动时获取
      - 集中化管理配置文件
      - 权限管理
      - 审批流程
    - 开发管理体系
      - 实时变更
      - 集中式下发
      - 灰度发布
      - 配置回滚
      - 配置生效监控 *
11.1.2 config
- 服务端和客户端组成
- 分布式系统
- 不依赖注册中心
- 存储配置信息的形式：jdbc, vault,native,svn,git（默认）
2 git版工作原理
- 配置中心服务器端，拉取本地git仓库的配置文件
- 本地git仓库同步远程git仓库配置文件
- 客户端拉取配置中心配置文件

11.1.3 config 入门案例
1 config server配置
- 1）创建maven工程
- 2）父级配置依赖---后续的子模块不用再添加相同的依赖
  - spring- boot- starter- test
  - spring- boot- starter- actuator
  - spring- boot- starter- web
- 3）创建工程模块
  - 创建一个module,config-server
- 4）配置依赖
  - pom 中添加config-server的依赖
    - spring- boot- starter- actuator
    - spring- cloud- config- server
- 5）编写主程序入口代码
    - 启动类：ConfigGitApplication类
    ```java
    @SpringBootApplication
    @EnableConfigServer // 启动spring cloud config服务功能
    public class ConfigGitApplication {
      public static void main(String[] args) {
        SpringApplication.run(ConfigGitApplication.class,args);
      }
    }
    ```
 - 配置文件配置：application.yml
```yaml

spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/zhongzunfa/spring-cloud-config.git
          username:
          password:
          search-paths: SC-BOOK-CONFIG
application:
  name: sc-config-git
server:
  port: 9090

```




















11.2 刷新配置中心信息
11.2.1 手动刷新操作
11.2.2 结合spring cloud bus 热刷新

11.2 本章小结
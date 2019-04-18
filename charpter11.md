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
 - 6）配置文件配置：application.yml
```yaml

spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/zhongzunfa/spring-cloud-config.git # git 服务器地址
          username: # git用户名
          password: # git 密码
          # 搜索SC-BOOK-CONFIG 目录下所有满足条件的配置文件，
          # 多个目录目录下所有满足条件的配置文件，多个用逗号隔开
          search-paths: SC-BOOK-CONFIG
application:
  name: sc-config-git
server:
  port: 9090

```
  - 在git仓库创建文件夹SC-BOOK-CONFIG，创建三个文件：config-info-dev.yml, config-info-test.yml, config-info-prod.yml,
  - config-info-dev.yml内容：
  ```yaml
  cn:
    springcloud:
      book:
        config: I am the git configura file from dev environment.
  ```
  - 映射关系：
    - application/profile/label
    - 应用名称(config-info)/激活环境名(dev，test,prod)/git分支(默认master)
  - 测试：http://localhost:9090/config-info/dev/master
  - config-server控制台打印信息可知，在本地的临时目录下面克隆了远程仓库的配置信息。

- 2 config client 配置
  - 1）创建工程模块
    - config-client 模块
  - 2）配置依赖 ： client 和web依赖
    - spring- cloud- config- client：客户端用来向服务端拉取配置所需依赖。
  - 3）编写主程序入口代码
    - 启动类ClientConfigGitApplication
    - 创建一个controller更好观察拉取到的git配置
    - 创建一个读取配置的对象
      - @ConfigurationProperties(prefix = "cn.springcloud.book")
  - 4）配置文件配置
    - bootstrap.yml文件优先于application.yml
    - 修改git配置，需要重启，下面有不需要重启的方式
    - application.yml
  ```yaml
  server:
    port: 9091
  spring:
    application:
      name: ch11-1-config-client
  ```
    - bootstrap.yml
    ```yaml
    spring:
      cloud:
        config:
          label: master # 代表请求那个git分支
          uri: http://localhost:9090 # 代表请求的config server 地址
          name: config-info # 代表请求那个应用名，也可以理解为git上面的文件名。
          profile: dev #那个环境的dev, test, prod
    ```

11.2 刷新配置中心信息
11.2.1 手动刷新操作
- 不需要重启，就可以手动刷新获取最新配置
- 1）配置依赖
  - spring-cloud-config-client
  - spring-boot-starter-security # 进行权限过滤，不进行端点拦截
- 2）配置文件配置
  - 再创建application.properties
    - management.endpoints.web.exposure.include=* //表示包含所有端点。
      - 默认情况下只打开info,health端点。
    - management.endpoints.health.show-details=always // 表示详细信息的显示
  - 修改application.yml文件
  ```yaml
    server:
      port: 9093
  ```
- 3)添加安全配置 config-client-refresh
```java
// SecurityConfiguration.java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable(); // 关闭端点的安全校验
  }
}
```
- 4）Controller类的变更
  - 在ConfigClientController上添加@RefreshScope注解
  - @RefreshScope注解：修饰的bean延迟加载，只有第一次访问时，才会被初始化。刷新bean，下次访问时会创建一个新的对象。
  ```java
  @RefreshScope
  @RestController
  public class ConfigClientController {
    @Autowired
    private ConfigInfoProperties configInfoValue;

    @RequestMapping("/getConfiginfo")
    public String getConfiginfo(){
      return configInfoValue.getConfig();
    }
  }

  @Component
  @RefreshScope
  public class ConfigInfoProperties {
    @Value("${cn.springcloud.book.config}")
    private String config;
  }
  ```
  - 修改远程仓库config-info-dev.yml配置，
  - 访问：localhost:9093/configConsumer/getConfigInfo发现内容没有变化。
  - 刷新配置信息，localhost:9093/actuator/refresh, 再访问上面的url，信息发生变化。
  -
11.2.2 结合spring cloud bus 热刷新
- 远程仓库通过git web hook通知config-server配置文件的变化
- config-server接收git的消息，spring cloud bus将消息发送到config client.
- cofig client接收到信息会重新发送加载配置信息的请求。
- 例子：使用Rabbit MQ作为消息中间件。

- 1）创建maven工程:依赖
  - spring-boot-starter-web
  - spring-boot-starter-actuator
  - spring-cloud-starter-bus-amqp

- 2) 创建module: config-server-bus 配置依赖
  - spring-cloud-starter-bus-amqp
  - spring-cloud-config-server
  - spring-boot-starter-security //简单的安全和端点开放
- 3）编写主程序入口代码
  - config-server-bus的启动类
  ```java
  @SpringBootApplication
  @EnableConfigServer
  public class GitConfigServerApplication {
    public static void main (String[] args) {
      SpringApplication.run(GitConfigServerApplication.clss, args);
    }
  }
  ```
  - 添加权限开放的配置
    ```java
    @Configuration
    public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
      @Override
      protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable(); // 关闭端点的安全校验
      }
    }
    ```
- 4) 配置文件配置
  - application.properties文件
    - management.endpoints.web.exposure.include=* //表示包含所有端点。默认情况下只打开info,health端点。
    - management.endpoints.health.show-details=always // 表示详细信息的显示
  - application.yml
  ```yaml
  spring:
    cloud:
      config:
        server:
          git:
            uri: htpps://gitee.com/zhongzunfa/spring-cloud-config.git
            serarch-paths: SC-BOOK-CONFIG
    application:
      name: config-server-bus
    ## 配置rabbitMQ信息
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest  
  server:
    port: 9090    
  ```
  - 热刷新的服务端创建完成，下面创建客户端。
- 5) 创建模块module,config-client-bus-refresh

- 6）配置依赖
  - spring-cloud-config-client
  - spring-boot-starter-security # 简单的安全和端点开放
- 7）启动类ClientConfigGitApplication
```java
  @SpringBootApplication
  public class GitConfigServerApplication {
    public static void main(String[] args) {
      SpringApplication.run(GitConfigServerApplication.class, args);
    }
  }
```
  - 创建controller用于查看远程git上的信息。
  ```java
    @RefreshScope
    @RestController
    @RequestMapping("configConsumer")
    public class ConfigClientContrller {
      @Autowired
      private ConfigInfoProperties configInfoValue;

      @RequestMapping("/getConfigInfo")
      public String getConfigInfo() {
        return configInfoValue;
      }
    }
  ```
- 8) 配置文件配置
  - bootstrap.yml
  ```yaml
  spring:
    cloud:
      config:
        label: master
        uri: http://localhost:9090
        name: config-info
        profile: dev
  ```
  - application.yml
  ```yaml
  server:
    port: 9095
  spring:
    application:
      name: config-client-bus-refresh  
  ```

- 测试：提叫git修改配置后，执行服务端的端点bus-refresh访问。
- 地址：http://local-host:9090/actuator/bus-refresh
-
```txt
执行服务端的url:http://local-host:9090/actuator/bus-refresh?destination=**刷新所有客户端,
可以把这个地址配置在webhooks上面，提交push文件之后，自动执行刷新的动作。
````























11.2 本章小结

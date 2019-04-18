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
### 12.5.3 客户端的安全认证机制JWT
1) 客户端发送请求，带上用户名和密码
2) 服务端返回JWT token
3) 客户端查询服务器端配置需要在header中带上token

- 1) 配置依赖
  - spring-boot-autoconfigure
  - spring-boot-configuration-processor
  - spring-cloud-starter-config
- 2) config 配置类
````java
@Configuration
@Order(Order.LOWEST_PRECEDENCE)
public  class ConfigClientBootstrapConfiguration {
  private static Log logger = LogFactory.getLog(ConfigClientBootstrapConfiguration.class);

  @Value("${spring.cloud.config.username}")
  private String jwtUserName;

  @Value("${spring.cloud.config.password}")
  private String jwtPassword;

  @Value("${spring.cloud.config.endpoint}")
  private String jwtEndpoint;

  private String jwtToken;

  @Autowired
  private ConfigurationEnviroment environment;

  // Spring实例化该Bean之后马上执行此方法，之后才会去实例化其他Bean，并且一个Bean中@PostConstruct注解的方法可以有多个。
  // @PostConstruct注解Bean中的某些方法，可以用在服务器启动时的做一些初始化工作。
  @PostConstruct
  public void init(){
    RestTemplate restTemplate = new RestTemplate();
    LoginRequest loginBackend = new LoginRequest();
    loginBackend.setUsername(jwtUserName);
    loginBackend.setPassword(jwtPassword);

    String serviceUrl = jwtEndpoint;
    Token token;

    try {
      //参数：请求地址，请求参数对象，接收返回对象
      token = restTemplate.postForObject(serviceUrl,loginBackend, Token.class);
      if (token.getToken() == null) {
        throw new Exception();
      }
      // 设置token
      setJwtToken(token.getToken());
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
  public String getJwtToken() {
    return jwtToken;
  }
  public void setJwtToken() {
    this.jwtToken = jwtToken;
  }

  @Bean
  public ConfigServicePropertySourceLocator configServicePropertySourceLocator(ConfigClientProperties configClientProperties) {
    ConfigServicePropertySourceLocator configServicePropertySourceLocator = new ConfigServicePropertySourceLocator(configClientProperties);
    configServicePropertySourceLocator.setRestTemplate(customRestTemplate());
    return configServicePropertySourceLocator;
  }

  @Bean
  public ConfigClientProperties configClientProperties(){
    ConfigClientProperties client
  }
}
```


























## 12.6 高可用部分
## 12.7 spring cloud 与appolo配置使用
## 12.8 spring cloud 与appolo结合使用实践
## 12.9 本章小结

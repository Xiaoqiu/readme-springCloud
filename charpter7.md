charpter7 zuul

7.1 zuul概述

- 认证与鉴权
- 压力控制
- 金丝雀测试
- 动态路由
- 负载削减
- 静态响应处理
- 主动流量管理
- 

例子：

- eruka工程

- zuul server工程

  - @EnableZuulProxy

  - @EnableDiscoverClient

  - ```yml
    spring:
    	application:
    		name: zuul-server
    server:
    	port: 5555
    eureka:
    	client:
    		serviceUrl:
    			defaultZone:   http://127.0.0.1:8888/eureka/
    zuul:
    	routes:
    		client-a:
    			path: /client/**  # 所有/client开头的映射到client-a这个服务
    			serviceId: client-a
    ```

  - 4 创建普通下游服务

  - ```yaml
    server:
    	port: 7070
    spring:
    	application:
    		name: client-a
    eureka:
    	client:
    		serviceUrl:
    			defaultZone: http://127.0.0.1:8888/eureka/
    	instance:
      	prefer-ip-address: true # 默认情况eureka会使用hostname进行服务注册，如果使用IP地址方式，使用这个参数。
    ```

  - 测试：

    - 通过Zuul调用调用地址: http://localhost:5555/client/add?a=100&b=400
    - 直接调用: http://localhost:7070/add?a=100&b=400
    - 

7.3 典型配置

7.3.1 路由配置

- 1 路由配置简化与规则

  - 1）单实例serviceId映射

  - ```yaml
    ## 方式一，原来配置
    zuul:
    	routes:
    		client-a:
    			path: /client/**
    			serviceId: client-a
    ## 方式二, 简化
    zuul:
    	routes:
    		client-a: /client/**
    ## 方式三，更加简化
    zuul:
    	routes:
    		client-a:
    ## 方式三 相当于添加一个/client-a/**的映射规则
    zuul:
    	routes:
    		client-a:
    			path: /client-a/**
    			serviceId: client-a
    
    ```

  - 2）单实例url映射 ： 路由到物理地址

    - ```yaml
      zuul:
      	routes:
      		client-a:
      			path: /client/**
      			url: http://localhost:7070 # client-a地址
      ```

    - 3） 多实例路由：使用Eureka中集成的基本负载均衡功能，而不是用Ribbon的负载均衡，如果想用Ribbon的负载均衡就要，禁止Ribbon使用Eureka, 直接配置Ribbon的负载均衡策略。

    - ```yaml
      zuul:
      	routes:
      		ribbon-route:
      			path: /ribbon/**
      			serviceId: ribbon-route # 指定一个serviceId
      			
      ribbon:
      	eureka:
      		enabled: false #禁止Ribbon使用Eureka
      		
      ribbon-route:
      	ribbon:
      		NIWSServerListCladdName: com.netflix.loadbalancer.ConfigurationBasedServerList
      		NFLoadBalancerRuleClassName: com.netflix.loadbalacer.RandomRule # 配置Ribbon的LB策略
      		listOfServers: localhost:7070, localhsot:7071 # 服务提供者
      		
      ```

      

    - 4）forward本地跳转： 在Zuul server里面写Controller逻辑处理代码，跳转到Zuul server的本地接口，就需要做forward配置。

      - ```java
        @RestController
        public class TestController {
          @GetMapping("/client")
          public String add(int a, int b){
            return "本地跳转：" + (a + b);
          }
        }
        ```

      - ```yaml
        zuul:
        	routes:
        		client-a:
        			path: /client/**
        			url: forward:/client
        ```

      - 5) 相同路径的加载规则：根据yaml原则，后面的覆盖前面的。所以最后一条配置生效

      - 

- 2 路由通配符
  - /**  匹配任意数量的路径与字符：
    - /client/add/a
  - /* 匹配任意数量的字符
    - /client/add
  - /? 匹配单个字符 
    - /client/a

7.3.2 功能配置

- 1 路由前缀
- 2 









































































































 
























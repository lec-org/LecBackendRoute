# SpringCloud

**Tips:** 极其不推荐 尚硅谷 2020年Spring Cloud 课程。建议直接看Spring Cloud Alibaba

## 微服务

微服务: 是一种架构思想, 不是框架. 微服务架构系统是一个分布式的系统, 按业务领域划分为**独立服务单元**, 有自动化运维 容错, 快速演进的特点.

**总结定义**: 将大应用拆分多个小模块, 有自己的功能和职责.


持续交付: ![](http://oss.re1ife.top/media/20230116153158.png)

- 不足
	- 微服务复杂度
	- 分布式事务问题
	- 服务划分分工
	- 服务部署- 自动化部署

- 微服务设计规则
### 单体架构
1. 业务越来越复杂, 繁体应用代码量大, 可读 维护 扩展性下降
2. 用户越来越多 程序承受并发越来越高
3. 测试难度越来越大

### SpringCloud简介

Java的微服务框架, 依赖于Spring Boot.

提供了很多组件帮助我们管理模块.

核心开发者: ![](http://oss.re1ife.top/media/20230116163658.png)

- 版本对应关系:
GA: 稳定版
目前H版和2022 都是稳定版本.
不同版本对于SpringBoot版本有要求

- 常用组件
- **服务注册和发现**: eureka: 停止更新 ,  **nacos（重要）,** consul
- 负载均衡: ribbon,dubbo
- 服务相互调用: openFeign,dubbo
- 服务容错: （x）hystrix, **sentinel**，**resilience4j**
- 服务网关: **gatewa**y, zuul
- 服务配置的统一管理: config-server, **Nacos**
- 服务消息总线: bus, **Nacos**
- 服务安全组件: security,Oauth2.0
- 服务监控: admin , jvm
- 链路追踪sleuuth+zipkin


- 总结
 SpringCloud 是微服务具体实现的一种方式.
实现方式有三种:
1. Dubbo+Zookeeper 半自动化的微服务架构
2. SpringCloud Netflix: 一站式微服务架构
3. SpringCloud Alibaba: 新一站式微服务架构

## Pom
- POM packaging 用于管理子模块依赖版本
- `<DependcyManagement>` 只是声明依赖并**不引入**，子模块需要显示声明的依赖而不需要版本号等
	- 有利于版本升级

## Eureka 服务注册与发现组件(x)

### 与Zookeeper区别

- 什么是**CAP**原则?
	-  一致性 (Consistency): 多个机器中的数据是一致的
	- 可用性 (A) : 当有一个节点宕机, 整个集群可以继续对外提供服务
	- 分区容错性性质(P) : 由于机房网络 或分区等原因导致各个机器中的数据短暂不一致(此特性不可避免)
	- CAP原则是指: 三个要素最多同时实现两个, 不可兼得.
		- AP : 数据可能不一致
		- CP: 数据一致但是有节点宕机, 整个服务不可提供服务
- 为什么服务注册中心不应该使用Zookeeper? 
	- Zookeeper 采用CP原则, 会导致服务暂时无法使用.
	- Eureka 注重AP高可用


- 其他注册中心:
	- Consul
	- Nacos



 ### Eurka 快速入门 

- 注册中心搭建
Eureka-server : 让其他组件注册 也可以自己注册自己
	- Eureka-clinet-a: 客户端
![](http://oss.re1ife.top/media/20230116175059.png)

#### Eureka-server搭建

- 导入依赖: 注意和`SpringCloud`版本一致
```xml
<dependency>  
   <groupId>org.springframework.cloud</groupId>  
   <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  
</dependency>
```

- 修改默认端口等配置:
```yaml
server:   
    port: 8761 # eureka 默认端口  
spring:  
    application:   
        name: eureka-server #应用名称
```
- 在启动类上添加`@EnableEurekaServer`的注解
- 运行之后 Eureka-server  控制台:
![](http://oss.re1ife.top/media/20230116183229.png)

#### Eureka-client
- 引入依赖
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
```
- 配置文件
```yaml
server:  
  port: 8080 # 客户端端口没有要求  
  
spring:  
  application:  
    name: eureka-client-a  
  
#注册 将自己的信息发给注册信息.  
eureka:  
  client:  
    service-url: #指定注册地址  
      defaultZone: http://localhost:8761/eureka
```

- 在启动类上添加`@EnableEurekaClient`的注解
- 运行示意图:
![](http://oss.re1ife.top/media/20230116210251.png) ![](http://oss.re1ife.top/media/20230116210251.png)


复制项目直接改端口:`--server.port:`
![](http://oss.re1ife.top/media/20230116211416.png)


#### Eureka配置文件

- server端配置详解
1. 注册中心服务列表(容器) 保存应用信息
3. 主动下线
4. 被动下线 注册中心剔除
5. 心跳机制- 注册中心检测应用是否够存活
	-  每次心跳就是一个请求 -> 剃掉
6. 应用每隔一段时间将注册中心的服务列表保存到本地 如何解决读脏数据?
7. 一段时间段内大量应用断开联系怎么办?
	- eureka-server不会剔除任何一个服务, 宁可放过一万不可错杀一个AP


```yml
  
# 配置分为 server client 实例 server 既是客户又是服务端  
eureka:  
  server:  
      eviction-interval-timer-in-ms: 100000 # 服务段每间隔 %d ms 做定期剔除服务操作  
      renewal-percent-threshold: 0.85 #续约百分比 超过85%的服务没有发出心跳 eureka版本保护服务 不会剔除任何一个服务  
      enable-self-preservation: true # server自我保护机制 防止网络原因造成的错误删除  
  instance: # 实例配置  
      instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port} # hostname: applicationName: port  
      hostname: localhost #主机名称或IP  
      prefer-ip-address: true # 以IP形式显示具体服务信息  
      lease-renewal-interval-in-seconds: 5 #服务实例的续约时间间隔
```

运行示意图
![](http://oss.re1ife.top/media/20230116215328.png)

- client 配置文件

```yml
server:  
  port: 8081  
spring:  
  application:  
    name: eureka-client-b  
eureka:  
  client:  
    service-url: # 注册地址  
      defaultZone: http://localhost:8761/eureka  
    register-with-eureka: true # 可以不向 eureka-server 注册  
    fetch-registry: true  # 是否拉取服务列表到本地  
    registry-fetch-interval-seconds: 10 # 为了缓解服务列表脏读问题 时间越短脏读越少 性能消耗越大  
  
  instance:  
    hostname: localhost # 主机名称 最好是ip 或者服务  
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port}  
    prefer-ip-address: true  
    lease-expiration-duration-in-seconds: 10  #续约时间
```


#### Eureka 集群


- 集群化方案
	- 中心化集群
	- 主从模式集群 
	- 去中心化模式: 更高可用 不需要中心服务
		- 在去中心化中没有主从概念

eureka 会将数据广播扩散, 也就是说eureka 集群采用的是去中心化, 因此`server`可以注册自己.
因此`注册中心集群`中各个节点是互相联系的


因此至少需要3个 `eureka-server`
- serverA
```yaml
server:  
  port: 8761  
spring:  
  application:  
    name: eureka-server  
  
# 如果不写 默认往自己里面注册  
eureka:  
  client:  
    service-url:  
      defaultZone: http://localhost:8762/eureka,http://localhost:8763/eureka  
  instance:  
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port}  
    hostname: localhost  
    prefer-ip-address: true  
    lease-renewal-interval-in-seconds: 5
```
- serverB
```yml
server:  
  port: 8762  
spring:  
  application:  
    name: eureka-server  
  
eureka:  
  client:  
    service-url:  
      defaultZone: http://localhost:8761/eureka,http://localhost:8763/eureka  
  instance:  
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port}  
    hostname: localhost  
    prefer-ip-address: true  
    lease-renewal-interval-in-seconds: 5
```
- serverC
```yml
server:  
  port: 8673  
spring:  
  application:  
    name: eureka-server  
  
eureka:  
  client:  
    service-url:  
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka  
  instance:  
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port}  
    hostname: localhost  
    prefer-ip-address: true  
    lease-renewal-interval-in-seconds: 5
```

如何搭建? 只需要设置配置文件

注册成功:
![](http://oss.re1ife.top/media/20230116223746.png)

但是集群还没有搭建成功, 应该怎么设置呢?

我们三个机器都是在本地, 它自己就认为我们是副本了.

我们还要修改 host 文件, 来骗他. 添加:
```
127.0.0.1 peer1  
127.0.0.1 peer2  
127.0.0.1 peer3
```


集群部署成功: 
![](http://oss.re1ife.top/media/20230116224650.png)

- 服务注册测试
clinetB测试注册
```yaml
server:  
  port: 8081  
spring:  
  application:  
    name: eureka-client-b  
eureka:  
  client:  
    service-url: # 注册地址  
      defaultZone: http://peer1:8761/eureka  
    register-with-eureka: true # 可以不向 eureka-server 注册  
    fetch-registry: true  # 是否拉取服务列表到本地  
    registry-fetch-interval-seconds: 10 # 为了缓解服务列表脏读问题 时间越短脏读越少 性能消耗越大  
  
  instance:  
    hostname: localhost # 主机名称 最好是ip 或者服务  
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port}  
    prefer-ip-address: true  
    lease-expiration-duration-in-seconds: 10 #续约时间
```

各个注册节点都是同步了的

- 集群终极配置方案

```yaml
server:  
  port: 8671
spring:  
  application:  
    name: eureka-server  
  
eureka:  
  client:  
    service-url:  
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka,http://localhost:8762/eureka
  instance:  
    instance-id: ${spring.application.name}:${server.port}  
    prefer-ip-address: true  
    lease-renewal-interval-in-seconds: 5
```

根据上面的配置, 只需要修改端口就好了.
而客户端也可以同时往三个节点同时注册, 放置集群其中一个宕机.




#### 集群深入理解

- 主从模式:
	-  主机如何选择
		- redis: 自己选择主机 
		- 哨兵模式: 主机挂了从从机中选择一个, 使用投票机制
			- 分布式数据一致性协议: Paxos raft (也用来选举)
			- 投票机制:  https://thesecretlivesofdata.com/
	- 数据如何同步
- CLUSTER


### Eureka 作用
![](http://oss.re1ife.top/media/20230220192656.png)
![](http://oss.re1ife.top/media/20230220200913.png)


### 调用Eureka查询注册服务

`DiscoveryClinet`查找注册的服务
```java
@RestController
@Slf4j
public class PaymentController
{
    @Value("${server.port}")
    private String serverPort;

    @Resource
    private PaymentService paymentService;

    @Resource
    private DiscoveryClient discoveryClient;

    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment)
    {
        int result = paymentService.create(payment);
        log.info("*****插入操作返回结果:" + result);

        if(result > 0)
        {
            return new CommonResult(200,"插入成功,返回结果"+result+"\t 服务端口："+serverPort,payment);
        }else{
            return new CommonResult(444,"插入失败",null);
        }
    }

    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id)
    {
        Payment payment = paymentService.getPaymentById(id);
        log.info("*****查询结果:{}",payment);
        if (payment != null) {
            return new CommonResult(200,"查询成功"+"\t 服务端口："+serverPort,payment);
        }else{
            return new CommonResult(444,"没有对应记录,查询ID: "+id,null);
        }
    }

    @GetMapping(value = "/payment/discovery")
    public Object discovery()
    {
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            System.out.println(element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance element : instances) {
            System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
                    + element.getUri());
        }
        return this.discoveryClient;
    }
}

```
### @LoadBalanced
负载均衡 还能自动替换请求URL

## Consul

HashiCorp 公司用GO开发。
提供微服务系统的服务治理、配置中心、控制总线的功能。可以根据需要选择使用， 也可全部使用。

优点：
- 健康度检查、HTTP、 DNS协议
- 支持跨数据中心WAN集群、图形界面
- 跨平台

docker安装：
	1. docker pull consul:latest
	2. docker run -d --net=host  consul agent -dev

Maven依赖：
```xml
<!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
```
服务提供者配置文件:
```yaml
server:
  port: 8006
spring:
  application:
    name: consul-provider-payment
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        heartbeat:
          enabled: true #心跳检测

```

服务主启动类：
```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class,args);
    }                                                
}
```



#### 异同点
|组建名|语言|CAP|服务健康检查|对外接口暴露|Spring Cloud集成|
|  :-:  | :-: | :-: | :-: | :-: | :-: |
|Eureka | Java| AP |可配支持| HTTP|集成|
|Consul | Go |CP | 支持|HTTP/DNS|集成|
|Zookeeper | Java | CP | 支持 | 客户端 |集成|




## Ribbon


- LB 负载均衡： 将用户的请求平坦的分配到多个服务上，达到系统的高可用
- 与Nginx区别
	- Nginx是服务器负载均衡
	- Ribbon是本地负载均衡， 调用微服务的时候从注册中心获取信息到本地， 实现PRC远程服务调用。
- 集中式LB: 在服务的消费方和提供方之间使用独立的LB设施。由该设施负责把访问请求通过某种策略转发到服务提供方。
- 进程内LB： 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器
	- Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

### 导入Ribbon
eureka-clinet自带
```yaml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
```  

直接导入
```yaml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>  
</dependency>
```


### Ribbon 核心组建IRule
![](http://oss.re1ife.top/media/20230222123714.png)


- IRule ： 通过特定算法从服务列表中选取一个访问的服务
	- RoundRobinRule ： 轮询
	- RandomRule：随机选一个
	- RetryRule ：先轮询，若获取服务失败指定时间内重试
	- WeightedResponseTimeRule：轮询扩展， 响应速度快的的越容易被选中
	- BestAvailableRule： 先过滤掉多次访问故障处于断路器跳闸状态服务， 选择并发量小的
	- AvailabilityFilteringRule：过滤故障，选择并发小的
	- ZoneAvoidanceRule： 默认规则，复合判断server区域和性能

#### 规则替换
- **官方明确警告**：
	- 自定义配置类不能处于`@ComponentScan`路径下，否则所有Ribbon客户端共享，无法**特殊化**
	- `@SpringBootApplication` 包含了上述注解
- 配置类， 启动类在`com.re1ife.atguigu_cloud2020`
```java
package com.re1ife.myrule;
@Configuration  
public class MyRule {  
    @Bean  
    public IRule randomRule(){  
        return new RandomRule(); //随机  
    }  
}
```

- 消费者启动类
```java
@SpringBootApplication  
@EnableEurekaClient  
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyRule.class)  
public class OrderMain80 {  
    public static void main(String[] args) {  
        SpringApplication.run(OrderMain80.class, args);  
    }  
}
```



### 负载均衡算法

#### 原理
- 负载均衡算法 = rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标
	- 每次服务重启后rest接口计数从1开始

#### 手写负载算法
//TODO
- 原理+JUC（CAS+ 自旋缩）



## OpenFeign

- 声明式 WebService客户端
- 功能：
	- Ribbon+RestTemplate
	- 进一步封装， 由他实现
	- 也集成Ribbon.与RIbbon 不同通过Feign只要定义服务绑定接口且以声明式方法，简单优雅。

微服务调用接口 + `@FeignClient`
- 引入
```xml
<!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

- 配置文件
```yaml
server:
  port: 8080
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka
```

- 主启动类
```java
@SpringBootApplication  
@EnableFeignClients  
public class OrderFeignMain8080 {  
  
    public static void main(String[] args) {  
        SpringApplication.run(OrderFeignMain8080.class, args);  
    }  
}
```

Service
```java
@Component  
@FeignClient("CLOUD-PAYMENT-SERVICE")  //对应的服务的id
public interface PaymentFeignService {  
  
    @GetMapping("/payment/get/{id}")   //远程服务的的接口 不是本地的
    public CommonResult<Payment> getPaymentById(@Param("id") Long id);
  
}
```
Controller
```java
@RestController  
@Slf4j  
public class OrderFeignController {  
    @Resource  
    private PaymentFeignService paymentFeignService;  
  
  
    @GetMapping("/consumer/payment/get/{id}")  
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){  
        return paymentFeignService.getPaymentById(id);  
    }  
  
}
```

然后就可以通过这个消费者向外暴漏的接口调用服务端的接口


#### OpenFeign 超时控制

OpenFeign 底层是Ribbon默认等待1S. 新版本没有

![](http://oss.re1ife.top/media/20230222190715.png)

配置中可以修改实际值：

旧版本配置，新版本OpenFeign Idea不会提示但是依然可用
```yml
ribbon:  
  # 建立之后从服务器读取可用资源时间  
  ReadTimeout: 5000  
  # 建立连接所用的最大时间  
  ConnectTimeout: 5000
```


新版本配置：
```yaml
feign: 
  client:  
    config:  
      default:  
        connect-timeout: 5000  
        read-timeout: 5000
```

### 日志打印功能

- 日志级别：
	- None 默认不显示任何日志
	- BASIC 仅记录请求方法、url、响应状态码及执行时间
	- HEADERS BASIC+请求和响应头信息
	- FULL HEADERS + 请求响应正文及元数据

1. 添加配置类
```java
@Configuration  
public class FeignConfig {  
    @Bean  
    Logger.Level feignLoggerLevel(){  
        return Logger.Level.FULL;  
    }  
}
```



# Gateway 网关

网关是所有微服务的入口
![](http://oss.re1ife.top/media/20230223134103.png)

- Gateway基于`异步非阻塞模型`
	- 异步非阻塞 Servlet3.1支持
- 基于Spring Framework5 ， Project Reactor和SpringBoot2.0 进行构建
- 动态路由： 可以匹配任何请求属性
	- 可以对路由指定断言和过滤器
- 继承Hystrix断路器功能
- 继承Spring Cloud 服务发现功能
- 医院编写的断言和过滤器
- 请求限流
- 支持路径重写


![](http://oss.re1ife.top/media/20230223135806.png)

Web发来的请求通过一系列的断言和过滤去最终分配到不同的服务。

![](http://oss.re1ife.top/media/20230223140617.png)


Web请求 -> Gateway 然后`Gateway Handler Mapping` 找到与请求相匹配的路由， 将其发送到Gateway Web Handler

`Handler` 再通过指定过滤器链来将请求发送到我们实际的服务执行业务逻辑 -》 返回

过滤器之间用虚线分割，是应为过滤器可能在发送代理请求之前或之后执行业务逻辑

Filter 在 `pre`类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等
`post`可以响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用

POM 依赖导入：
```xml
<dependencies>  
    <!--gateway-->  
    <dependency>  
        <groupId>org.springframework.cloud</groupId>  
        <artifactId>spring-cloud-starter-gateway</artifactId>  
    </dependency>    <!--eureka-client-->  
    <dependency>  
        <groupId>org.springframework.cloud</groupId>  
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
    </dependency>    <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->  
    <dependency>  
        <groupId>com.atguigu.springcloud</groupId>  
        <artifactId>cloud-api-commons</artifactId>  
        <version>${project.version}</version>  
    </dependency>    <!--一般基础配置类-->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-devtools</artifactId>  
        <scope>runtime</scope>  
        <optional>true</optional>  
    </dependency>    <dependency>        <groupId>org.projectlombok</groupId>  
        <artifactId>lombok</artifactId>  
        <optional>true</optional>  
    </dependency>    <dependency>        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-test</artifactId>  
        <scope>test</scope>  
    </dependency></dependencies>
```


YAML
```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://127.0.0.1:7001/eureka
```

启动类：
```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GatewayMain9527.class, args);
    }
}

```
## 如何做路由映射
1. 找到想要映射的Controller选择对应的接口
2. 将9527端口暴露允许访问而隐藏8001
3. 编写对应规则：
```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://127.0.0.1:7001/eureka

```

**Gateway** 不要引入web包
测试结果
![](http://oss.re1ife.top/media/20230223143022.png)

## 路由配置方式

1. yaml 配置方法
2. 代码中注入`RouteLocator`的Bean
	1. 测试例子来访问 https://news.baidu.com/finance

```java
@Configuration
public class GateWayConfig {
    @Bean
    public RouteLocator customRouteLocator (RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes =  routeLocatorBuilder.routes();
        routes.route("path_route_atguigu",r -> r.path("/finance").uri("https://news.baiodu.com")).build();
    
        return routes.build();
    }
}

```
仔细对别YAML的配置可以发现， 其实相关的东西都是写到了的。
3. 动态路由
	1. 加入eureka-client依赖
	2. 配置文件编写 
```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka


```


## GATEWAY常用Predicate

Gateway 将路由匹配作为 Spring WebFlux HandlerMapping基础架构
包含许多内置Route Predicate 工厂， 所有这些Predicate都与http请求的不同属性匹配。
多个Route Predicate工厂组合。

Gateway通过 Route Predicate 工厂 创建断言对象， 断言对象可以发而u只给route

![](http://oss.re1ife.top/media/20230223160018.png)
1. 是设置什么时间点之后开始生效
	- yml![](http://oss.re1ife.top/media/20230223182223.png)
	- 可以通过`ZoneTime.now()`实现
2. 实在什么时间点之前有效 
3. 在什么时间段之间有效
4. 是否携带cookie（下图表示要有username=zzyy）
 ![](http://oss.re1ife.top/media/20230223220504.png)
5. 请求头
 ![](http://oss.re1ife.top/media/20230223220646.png)

6. 。。。。
7. 。。。。方法嘛
8. Path 就是path之前写过了
9. Query 支持传入两个参数 ， 一个属性名一个属性值（可为正则）

## Filte 路由过滤器
内置多种过滤器
- 生命周期
	- pro
	- post
- 种类
	- GatewayFilter ： 31个
	- GlobalFilter ： 10多个
- 常用过滤器
- ![](http://oss.re1ife.top/media/20230223221454.png)

- 自定义过滤器
 	1. 实现两个接口 GlobalFilter,Ordered
 	2. 可以干嘛： 全局日志记录， 统一网关鉴权，....
```java
@Configuration  
@Slf4j  
public class MyLogGateWayFilter implements GlobalFilter, Ordered {  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
        log.info(("-----come in MyLogGateWayFIlter" + new Date());  
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");  
        if(uname == null){  
            log.info("*** 用户名为null, 非法用户");  
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);  
            return exchange.getResponse().setComplete();  
        }  
        return chain.filter(exchange);  
    }  
  
    @Override  
    public int getOrder() {  
        return 0;  
    }  
}
```
	 1.  测试
	无uname
![](http://oss.re1ife.top/media/20230223222400.png)

有uname
![](http://oss.re1ife.top/media/20230223222429.png)


# Sleuth 分布式请求链路跟踪

一个客户端请求在后端系统经过多个不同服务节点调用来产生最后的请求结果，形成一个复杂分布式服务调用链路， 中间一环出现高延时或错误都会失败。

- Spring Cloud Sleuth 提供完整的服务跟踪结局方案

![](http://oss.re1ife.top/media/20230224083322.png)

## 搭建步骤
1. Zipink
	1. 下载： SprinCloud F版本起不用自己构建Zipkin Server， 只需要调用jar
		1. https://repo1.maven.org/maven2/io/zipkin/zipkin-server/
2. 表示一请求链路，一条链路通过Trace Id唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来
![](http://oss.re1ife.top/media/20230224085144.png)
	流程图精简：
![](http://oss.re1ife.top/media/20230224085555.png)

- 名词解释： 
	- Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识
	- Span： 表示调用链路来源， 一次请求信息

## 实际测试

1. 服务提供者：cloud-provider-payment8001
	1. pom
		```java
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
		```
	2. yaml
	  ```yaml
	  spring:  
	    zipkin:  
	      base-url: http://localhost:9411  
	      sleuth:  
	        sampler:  
	          #采样率值介于 0 到 1 之间，1 则表示全部采集  
	          probability: 1
	  ```
		
	 ![](http://oss.re1ife.top/media/20230224090241.png)
2. 消费者也同上
	1. 添加Zikin的接口

```java
    @GetMapping("/payment/zipkin")
    public String paymentZipkin(){
        String result = restTemplate.getForObject("http://localhost:8001/payment/zipkin", String.class);
        return result;
    }
```

测试样例：
![](http://oss.re1ife.top/media/20230225131553.png)
详
![](http://oss.re1ife.top/media/20230225131803.png)

# SpringCloud Alibaba

[[SpringCloud Alibaba]]


Alibaba 的实现：
-   **流量控制和服务降级**：使用[阿里巴巴Sentinel进行](https://github.com/alibaba/Sentinel/)流量控制，断路和系统自适应保护
-   **服务注册和发现**：实例可以在[阿里巴巴Nacos中](https://github.com/alibaba/nacos/)注册，客户可以使用Spring管理的bean发现实例。通过Spring Cloud Netflix支持Ribbon，客户端负载均衡器
-   **分布式配置**：使用[阿里巴巴Nacos](https://github.com/alibaba/nacos/)作为数据存储
-   **事件驱动**：构建与Spring Cloud Stream [RocketMQ](https://rocketmq.apache.org/) Binder连接的高度可扩展的事件驱动微服务
-   **消息总线**：使用Spring Cloud Bus RocketMQ链接分布式系统的节点
-   **分布式事务**：支持高性能且易于使用的[Seata](https://github.com/seata/seata)分布式事务解决方案[](https://github.com/seata/seata)
-   **Dubbo RPC**：通过[Apache Dubbo RPC](https://dubbo.apache.org/en-us/)扩展Spring Cloud服务到服务调用的通信协议


Dashboard ：
- 默认用户名  nacos
- 默认  密码   nacos


> 官方文档： https://github.com/alibaba/spring-cloud-alibaba/blob/2022.x/README-zh.md

> 版本说明： https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

![](http://oss.re1ife.top/media/20230225151920.png)


- Mavne 依赖导入
- 最新版本Spring Cloud 要求JDK>=17
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.9.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

springBoot 2.x 
```JAVA
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.10-RC1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

可以直接用Docker 拉去
```bash
docker pulll nacos/nacos-server
```


本次教程全部采用 `2.2.10-RC1`
# Nacos 服务注册与配置中心


- 替换eureka config
- Naming Configuration

> 开源地址： https://github.com/alibaba/Nacos


## Discovery

![](http://oss.re1ife.top/media/20230225152853.png)


- 引入依赖 
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```


- 配置文件
```yaml
server:  
  port: 9001 # server port  
  
spring:  
  application:  
    name: nacos-payment-provider  
  cloud:  
    nacos:  
      discovery:  
        server-addr: 127.0.0.1:8848 # nacos 地址  
management:  
  endpoints:  
    web:  
      exposure:  
        include: '*'
```

**注意：** 如果不想使用Nacos作为注册发现可以将`spring.cloud.nacos.discovery`设置为`false`


- 启动类
```java
/**
 * @author re1ife
 * @data 2023-02-25 16:11:48
 */
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class, args);
    }
}

```

- controller
```java
@RestController  
public class PaymentController {  
    @Value("${server.port}")  
    private String serverport;  
  
    @GetMapping(value = "/payment/nacos/{id}")  
    public String getPayment(@PathVariable("id") Integer id){  
        return "nacos registry ,server port: "+ serverport +"\t id"+id;  
    }  
}
```

- 重复建造一个payment9002
- 服务名不同情况：
![](http://oss.re1ife.top/media/20230226131628.png)
- 相同：
- ![](http://oss.re1ife.top/media/20230226131725.png)

### 消费者注册和负载均衡

- Nacos 自动ribbon负载均衡                                                                                                                                                                                                                        

1. 消费者模块化创建
2. POM依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>        <groupId>com.re1ife.atiguifu_springcloud</groupId>  
        <artifactId>cloud2020</artifactId>  
        <version>1.0</version>  
    </parent>  
    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>  
  
    <properties>        <maven.compiler.source>11</maven.compiler.source>  
        <maven.compiler.target>11</maven.compiler.target>  
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
    </properties>    <dependencies>        <!--SpringCloud ailibaba nacos -->  
        <dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
        </dependency>        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->  
        <dependency>  
            <groupId>com.re1ife.atiguifu_springcloud</groupId>  
            <artifactId>cloud-api-commons</artifactId>  
            <version>1.0</version>  
        </dependency>        <!-- SpringBoot整合Web组件 -->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-actuator</artifactId>  
        </dependency>        <!--日常通用jar包配置-->  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>  
            <scope>runtime</scope>  
            <optional>true</optional>  
        </dependency>        <dependency>            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>    </dependencies></project>
```

- yaml 配置 
```yaml
server:  
  port: 8083  
  
  
spring:  
  application:  
    name: nacos-order-consumer  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848  
  
  
#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)  
service-url:  
  nacos-user-service: http://nacos-payment-provider 
 ```

**上面的Service-url** 对应
```java
//Eureka写法
public static final String SERVER_URL = "http://nacos-payment-provider";
```

- 消费者启动类
```java
@SpringBootApplication
@EnableDiscoveryClient //开启服务发现
public class ConusmerNacosMain83 {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```
- 配置类
```java
@Configuration  
public class ApplicationContextConfig {  
    @Bean  
	//LoadBalanced 会自动解析服务名的url
    @LoadBalanced    
    public RestTemplate getRestTemplate(){  
        return new RestTemplate();  
    }  
}
```

- controller

```java
package com.re1ife.atiguifu_springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

/**
 * @author re1ife
 * @data 2023-02-26 13:33:54
 */

@RestController
@Slf4j
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serviceURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id){
        return restTemplate.getForObject(serviceURL+"/payment/nacos/"+id,String.class);
    }


}

```


 ### 注册中心对比

- nacos 支持 AP或CP
- C ： 数据强一致性
- A： 高响应
- P分区容忍：系统中任意信息的丢失或失败不会影响系统的继续运作


## 服务配置中心

- 出现背景
微服务意味着将单体应用中业务拆分成子业务， **颗粒度**变小， 存在大量配置信息， 因此要集中管理配置服务。

- 服务端和客户端
	- 服务端：分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。
	- 客户端：通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器 （默认git）

- 替代Config

	Nacos 和 SpringCloud 一样， 在项目初始化的时候， 保证先从配置中心拉取配置。完成之后才可以正常启动

springboot 配置文件优先级： bootstrap > application
> 这里的优先级是指读取的优先级， 配置内容是低优先级覆盖高优先级




**Nacos 配置管理 `dataId`格式：**
```
${prefix}-${spring.profile.actice}.${file-extension}
```
-  `prefix` 默认为 `spring.application.name`值， 也可通过`spring.cloud.nacos.config.prefix`配置
-   `spring.profiles.active` 即为当前环境对应的 profile，可参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 
	- **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
-   `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。


### 配置客户端
- 依赖引入：
```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

- 搭建client
- bootstrap yaml
```yaml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}

```
- applicaton yaml
```yaml
spring:
  profiles:
    active: dev # 表示开发环境
```

> 结合 application和 bootsrap 就会去nacos config 找到 dev版本的yaml 配置文件

- 主启动类
```java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377
{
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
``` 


-  ConfigController
```java
@RestController
//使得当前类下的配置支持 Nacos 动态刷新功能
@RefreshScope
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```


**config.info报错注意：**
1. boot 新版本休要手动引入`spring-cloud-starter-bootstrap`
2. 检查`bootstrap.yaml`文件名是否写错了

手动跟新配置文件后， 动态刷新：
![](http://oss.re1ife.top/media/20230226154925.png)


### 配置中心分类配置



- 出现背景：多环境多项目管理
	- 一个系统会有 DEV PROD TEST环境 

Namespace、Group和Service区别：

![](http://oss.re1ife.top/media/20230226155445.png)
- 默认下： Namespace  = public ， Group = DEFAUL_GROUP , 默认Cluster= DEFAULT
- Namespace 主要用来实现隔离
	- 生产、开发、测试三个环境
- Group 是可以把不同的微服务划分到同一个分组
- Service 是微服务： 一个service 多个Cluster（集群）
- Cluster： 对指定微服务的一个虚拟划分。为了容灾可以把Service 微服务分别部署在不同机房
- Instance： 微服务的实例

#### DataId方案
主要是在 Nacos server 里面新建 test/prod 的配置文件。

然后通过更改 `spring.profile.active` 来进行修改。


#### Group 方案

1. Nacos控制台创建`DEV_GROUP`
 ![](http://oss.re1ife.top/media/20230226161804.png)
2. 修改client端的`bootstrap` 和 application
![](http://oss.re1ife.top/media/20230226161942.png)

 ![](http://oss.re1ife.top/media/20230226162002.png)

- 测试结果
![](http://oss.re1ife.top/media/20230226162036.png)

#### Namespace

大差不差。 

1. Nacos 控制台新建命名空间
2. 选中命名空间
3. 新建配置文件
4. 配置yaml
```yaml
spring:  
  application:  
    name: nacos-config-client  
  cloud:  
    nacos:  
      discovery:  
        server-addr: 127.0.0.1:8848 
      config:  
        server-addr: localhost:8848 
        file-extension: yaml 
        namespace: xxxxxx--xxxxx---xxxxx   ## 配置选则命名空间的id 
          group: DEV_GROUP
```


## 集群和持久化配置（Important）

- 前提条件
	- Linux
	- MySQL 主从模式 或者高可用数据库 
		- 默认情况下 Nacos使用了嵌入式数据库derby
			- 集群化部署 存在一致性问题
			- 仅仅支持MySQL的集中式存储

集群架构图：
![](http://oss.re1ife.top/media/20230226162818.png)
- VIP: Virtual IP 虚拟IP 通过NGINX集群 反向代理 -> Nacos集群

- Mysql 设置步骤：
 1. 安装数据库 5.65+
 2. 初始化数据库，初始化文件： nacos-mysql.sql
 3. 修改 `nacos` 的conf/application.yaml ， 增加支持mysql数据源配置
	 - ![](http://oss.re1ife.top/media/20230226191031.png)
 4. 添加mysql数据源url ， 用户名与密码 
 5. 再次已单机模式运行 nacos 转移数据
	 1. `start.sh -m standalone`
6. 配置Cluster 
7. `./start.sh -p xxxx` 以xxxx端口启动

配置集群文件：
- 配置 nacos/conf/Cluster
```conf
# ip:port
200.8.9.16:8848
200.8.9.17:8848
200.8.9.18:8848
```
 


- Nginx Conf 负载均衡
```conf
upstream nacos{
	server1 200.8.9.16:8848;
	server2 200.8.9.17:8848;
	server3 200.8.9.18:8848;
}


server {
	listen 1111;
	proxy_pass http://nacos/
}
```



# Sentinel

- 分布式流量来的防卫兵（Hystrix alibaba翻版）

Hystrix:
 - 手工搭建监控平台
 - 没有Web界面 进行细颗粒度的配置流量控制、速率控制、服务熔断...

Sentinel:
- 单独组件
- 界面化细颗粒度
- 目标尽量配置、少少写代码

![](http://oss.re1ife.top/media/20230227124307.png)

组成部分：
 - 核心库（Java客户端）： 不依赖于任何框架，可以运行在所有Java环境。 对Dubbo和Spring Cloud 支持良好
 - 控制台： 基于SpringBoot开发， 打包后直接运行。

Dashborad：
- 默认用户： sentinel
- 密码： sentinel


How to Use?
1. 运行Nacos server
2. 新建cloudalibab-sentinel-service8401模块
	1. Pom
```xml
<dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件+actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.3</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>
```
	2. Yml
```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #Nacos服务注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'
```
	3. 主启动类
```java
@EnableDiscoveryClient
@SpringBootApplication
public class SentinelMain8401 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelMain8401.class, args);
    }
}

```
	4. 业务类
```java

@RestController
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA(){
        return "I am test A";
    }
    @GetMapping("/testB")
    public String testb(){
        return "I am test B";
    }
}
```
2. 启动sentinel server
3. 启动微服务

**Tips：**
1. Sentinel 是懒加载， 服务启动时不会立马显示
![](http://oss.re1ife.top/media/20230227131735.png)
2. 执行一次他的接口之后才会添加
![](http://oss.re1ife.top/media/20230227132115.png)

## Sentinel 流控

- 资源名： 默认路径名称
- 针对来源： 针对调用者限流， 填写微服务名

- 阈值类型：
	- QPS（每秒请求数量）：QPS达到阈值限流
	- 线程数： 调用api的线程数达到阈值进行限流。

- 集群模式？：不需要

- 流控模式：
	- 直接： api达到限流条件直接限流
	- 关联： 关联资源达到阈值 限流自己
		- B关联的A资源到达阈值，限流A自己
		- ![](http://oss.re1ife.top/media/20230227181920.png)
 
	- 链路了只记录特定链路的流量（指定资源从入口资源进入的流量达到阈值）。,
	

- 流控效果
	- 快速失败： 直接失败抛异常
		- ![](http://oss.re1ife.top/media/20230227140431.png)
		- **要配置全局异常**

	- Warm up： 根据Code Factor冷加载因子，从阈值/codeFactor 经过预热时长，达到阈值QPS
		- 公式： 阈值 / coldFactor(默认3) 经过要预热时长后才可以达到阈值。 
		- 案例： 秒杀系统突然瞬间会有很大流量，预热可以慢慢把流量放进来，慢慢增长阈值。
	- 排队等待： 匀速排队，让请求匀速通过， 阈值的类型必须设置为QPS否则无效


## 熔断降级

- Tips： 熔断会降级，但是降级不一定熔断。
	- 熔断：在互联网系统中，当下游服务因访问压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。这种牺牲局部，保全整体的措施就叫做熔断。
	- 降级：当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作。
		- 就是尽量把资源给核心
	- 服务降级和熔断：
		- 相同点：
			- 目标相同： 可用性和可靠性
			- 用户体验：某些功能不能用
		- 不同：
			- 熔断一般是下游故障引起的
			- 服务降级是整体负荷

- RT(老版本)： 平均响应时间
	- RT超出阈值且时间窗口内请求 >= 5 触发降级
	- ![](http://oss.re1ife.top/media/20230227184459.png)
 
	- 窗口期过后关闭断路器
	- RT最大4900
- 慢调用比例：设置允许的慢调用 **RT（即最大的响应时间）**，请求的**响应时间**大于该值则统计为**慢调用**。当单位统计时长（`statIntervalMs`）内请求数目大于设置的**最小请求数目**，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

- 异常比例数（秒级别）：QPS >= 5 且超过阈值触发降级， 时间窗口结束关闭降级。
- 异常数（分钟）： 异常数超过阈值，触发降级，时间窗口结束关闭降级

- 半开状态：新版本有了
	- 自动检测是否有请求异常
		- 没有： 关闭断路器
		- 有：持续


## 自适应保护


- 目的
	- 保证系统可靠 不跨
	- 稳定前提下、保证吞吐量
- 原自适应保护根据硬指标->负载（Load）
	- 缺点： 
		- 根据负载Load调整流量，有延迟。
		- 恢复慢，下游不可靠可能会导致RT很高

- TCP BBR算法启发
	- [[计网#拥塞控制]]

- 系统规则：
	- Load （Unix-like 有效）：`load1`超阈值且并发线程超过系统容量触发保护
		- 系统容量: maxQPS \* minRt
	- cpu usage
	- RT
	- 线程数
	- 入口QPS


## 热点Key限流（重要且实用）

- 热点： 频繁访问的数据
	- 举例子： 商品Id, 一段时间内某个商品爆火
- 热点参数会统计传入参数中的热点参数，根据限流方式进行限流。
- http://xxxxxx?param=qqq, qqq 就是参数

`@SentinelResouce` : 处理的 Sentinel 控制台的违规，而不是 Java 的

例子：

```java
	@GetMapping("/hotKeyTest")
    //下面的注解value不需要和GetMapping的值一样，符合Sentinel的Key就好
    // blockHandler 是不符合热点规则的处理方法
    @SentinelResource(value = "hotKeyTest", blockHandler = "howDareU")
    public String hotKeyTest(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2){

        return "THIS IS A HOT KEY";
    }

    public String howDareU(String p1, String p2, BlockException exception){
        return "how dare you break the hot key law? Fuck  u!";
    }
```

![](http://oss.re1ife.top/media/20230227194648.png)

- 参数索引： 第几个参数

1.  value不需要和GetMapping的值一样，符合Sentinel的Key就好, 唯一
2. blockHandler 是不符合热点规则的处理方法

测试结果：
![](http://oss.re1ife.top/media/20230227194941.png)

### 参数例外项：
1. 期望P1 参数特殊值是，限流值变化
	- 只支持JAVA 八种基本类型

![](http://oss.re1ife.top/media/20230227195333.png)


p1="1"时：
![](http://oss.re1ife.top/media/20230227195318.png)

p1="2"

![](http://oss.re1ife.top/media/20230227195403.png)



## Resouces配置

引入之前自己的请求包。

### 资源名称限流

主要还是：``@SentinelResouce(value="资源名", blockHandler="兜底方法名")``、

### URL 限流
`@XXXMAPPING("url")`
`@SentinelResouce(value="资源名")`

流量控制选择url

上述方法的核心问题：没有全局处理兜底方法。

### 自定义限流逻辑

1. 自定义限流类
```java
public class CustomerBlockHandler
{
    public static CommonResult handleException(BlockException exception){
        return new CommonResult(2020,"自定义的限流处理信息......CustomerBlockHandler");
    }
}
```
2. 自定义限流处理逻辑
```yaml\
@GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            blockHandlerClass = CustomerBlockHandler.class, blockHandler = "handleException2")
    public CommonResult customerBlockHandler()
    {
        return new CommonResult(200,"按客户自定义限流处理逻辑");
    }
```


## 持久化

将限流配置持久化进入Nacos。

1. 持久化包以来
```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

2. YAML
```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持
```

3. nacos 新建配置
![](http://oss.re1ife.top/media/20230227200803.png)

配置解读：
```
resource：资源名称；

limitApp：来源应用；

grade：阈值类型，0表示线程数，1表示QPS；

count：单机阈值；

strategy：流控模式，0表示直接，1表示关联，2表示链路；

controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；

clusterMode：是否集群。
```
4. 规则出现


//Todo
# Seate
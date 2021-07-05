# 概念

## 什么是微服务

将单一的应用程序化成小的服务模块，每个服务模块都是独立的进程

将业务区分，分为一个个服务模块，每一个小服务可以拥有自己的数据库，它强调的时个体

## 微服务架构

个个微服务直接相互调用，形成一个架构，它强调的时整体

## dubble与cloud的区别

dubble是基于rpc远程调用，cloud时基于restful的

## 微服务的优缺点

### 优点

- 服务足够小

- 代码易于理解

- 开发效率高，一个服务只做一个事

- 有利于解耦

- 微服务能用不同语言开发 

- 微服务可以有自己的数据库，也可以由统一的数据库

### 缺点
- 增加了运维成本
- 使整体的服务变复杂

## SpringCloud是什么

SpringCloud=分布式微服务架构下的一站式解决方案，
是各个微服务架构落地技术的集合体，俗称微服务全家桶

# 建立公共模块-api

## 建立实体bean

**Lombok**

Lombok能以简单的注解形式来简化java代码，提高开发人员的开发效率。例如开发中经常需要写的javabean

引入jar包

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

```java
public class TUser {
    @Getter
    @Setter
    private String id;
    @Getter
    @Setter
    private String username;
}
```

## 搭建基础环境

这个是微服务提供者（mybatis+controller）：dept-8001

## 新建微服务消费者 

建立微服务 dept-80

配置config

```java
@Configuration
public class ConfigBean {

    @Bean
    public RestTemplate getRestTemplate(){
        //RestTemplate提供了多种便捷访问远程Http服务的方法
        return new RestTemplate();
    }
}
```

### 使用RestTemplate

RestTemplate提供了多种便捷访问远程Http服务的方法， 
是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集

```java
@RestController
public class UserController {
    @Autowired
    private RestTemplate restTemplate;
    private final String url = "http://localhost:8001";

    @GetMapping("getUser")
    public List getUser(){
        return restTemplate.getForObject(url+"/getUser",List.class);
    }
}
```

# Eureka服务注册与发现

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现(请对比Zookeeper)。

Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。SpringCloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。



Eureka包含两个组件：Eureka Server和Eureka Client
Eureka Server提供服务注册服务
各个节点启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到


EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）

## 三大角色

Eureka Server 提供服务注册和发现

Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到

Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务

**EurekaClient:通过注册中心进行访问没如果server多个心跳周期没有收到某个节点，则server会将其从服务注册表移除（默认90秒）**

## 建立eureka服务

新建cloud-eureka-7001服务

引入jar包

```xml
<dependencies>
    <!--eureka-server服务端 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
```

配置文件

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称，后续defaultZone通过这个来关联eureka
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，
    #我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

使用EurekaServer开启server

```java
@SpringBootApplication
// EurekaServer服务器端启动类,接受其它微服务注册进来
@EnableEurekaServer
public class Application7001 {
    public static void main(String[] args) {
        SpringApplication.run(Application7001.class,args);
    }
}
```

访问<http://localhost:7001/>

## 微服务注册

微服务注册进eureka服务中心

### 基本信息注册

将cloud-provider-dept-8001注册服务中心

添加依赖

```xml
<!-- 将微服务provider注册进eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

添加配置信息 

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
```

在启动类添加注解

```java
@SpringBootApplication
@MapperScan("com.xiao.mapper")
//本服务启动后会自动注册进eureka服务中
@EnableEurekaClient
public class Application {
```

测试：

先启动eureka 再启动 被注册的服务

访问：<http://localhost:7001/> 能看到注册信息

| Application | AMIs        | Availability Zones | Status                                                       |
| :---------- | :---------- | :----------------- | :----------------------------------------------------------- |
| **UNKNOWN** | **n/a** (1) | (1)                | **UP** (1) - [192.168.1.101:8001](http://192.168.1.101:8001/info) |

## 添加服务别名（status）

**添加application名称与status别名**

引入包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

配置文件添加

```yml
spring:
  application:
    name: provider-dept #添加Application名称，可以用与feign和rebbion访问
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: provider-dept-8001 #每个微服务提供者这个instance-id都必须不同（Status）
    #访问路径可以显示IP地址
    prefer-ip-address: true
```

结果：

| Application       | AMIs        | Availability Zones | Status                                                       |
| :---------------- | :---------- | :----------------- | :----------------------------------------------------------- |
| **PROVIDER-DEPT** | **n/a** (1) | (1)                | **UP** (1) - [provider-dept-8001](http://192.168.1.101:8001/info) |

## 添加springboot info信息

父工程添加

```xml
<build>
    <finalName>cloudparent</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <delimit>$</delimit>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

子工程配置，$开始和$结束，在src/main/resources下，就能取到maven配置文件的信息

```yml
info:
  app.name: xiao-microservicecloud
  company.name: www.xiao.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

## eureka自我保护

某时刻某一个微服务不可用了，eureka不会立刻清理，一定时间内（默认90秒）依旧会对该微服务的信息进行保存，这是CAP里面的AP思想

## eureka 的集群

修改host文件映射

127.0.0.1	eureka7001.com
127.0.0.1   eureka7002.com
127.0.0.1   eureka7003.com

新建两个eureka项目

修改yml配置

```yaml
eureka:
  instance:
    #hostname: localhost #eureka服务端的实例名称
     hostname: eureka7001.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，
    #我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      #defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

修改注册的服务配置

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

访问：<http://eureka7001.com:7001/>能看到集群信息

> Euraka 与zookeeper的区别

 在分布式系统领域有个著名的CAP定理

1. C-数据一致性；

2. A-服务可用性；

3. P-服务对网络分区故障的容错性(单台服务器，或多台服务器出问题（主要是网络问题）后，正常服务的服务器依然能正常提供服务)，这三个特性在任何分布式系统中不能同时满足，最多同时满足两个）

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/1.png)

**CAP理论也就是说在分布式存储系统中，最多只能实现以上两点。而由于当前网络延迟故障会导致丢包等问题，所以我们分区容错性是必须实现的。也就是NoSqL数据库P肯定要有，我们只能在一致性和可用性中进行选择，没有Nosql数据库能同时保证三点。（==>AP 或者 CP）**



RDBMS==>（MySql,Oracle,SqlServer等关系型数据库）遵循的原则是：ACID原则（A：原子性。C：一致性。I：独立性。D：持久性。）。

NoSql==>    （redis,Mogodb等非关系型数据库）遵循的原则是：CAP原则（C：强一致性。A:可用性。P：分区容错性）。

一般来说 p 是必须满足的，然后我们只能在c和a之间做选择

如在大型网站中，选择的时AP原则，因为服务不能挂，挂了就是很严重的灾难性事故



zookeeper保证的是cp：在向注册中心注册时，zookeeper可以允许几分钟的注册事件，但不能接收服务down掉不可用

，当master接口与其他节点失去联系时，其余节点重新选择leader，但如果选择leader时间太长，选举期间，整个zk集群是不可用的，这时就会导致注册服务瘫痪。

euraka保证的是ap：eureka各个节点平等，只要有一台在，就能保证注册服务，只不过查到的信息可能不是最新的，此外，它还有一种自我保护机制：在15分钟内，85%节点没有正常心跳，则eureka认为客户端与注册中心网络出现故障，则出现以下几点现象

1 不会从注册表移除没有心跳的过期服务

2 任然接受新的注册与查询，但不同步其他节点

3 网络稳定，再同步到其他节点

# Ribbon负载均衡

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端**的**负载均衡**的工具。

负载均衡分为两种：

集中式：

​	即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责	把访问请求通过某种策略转发至服务的提供方；

进程内LB：

​	将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合	适的服务器。

​	Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



Ribbon在工作时分成两步
第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server.
第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。
其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。



## Ribbon的基本实现

首先，它集成的是客户端的

在80端口上进行集成

导入jar包

**H.SR1版本之后，只需要引入类似spring-cloud-starter-zookeeper-discovery的jar包**

```xml
<!-- Ribbon相关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

配置文件

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    # 因为时客户端，所以不需要向注册中心注册
    register-with-eureka: false
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

config配置更改

```java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
    //RestTemplate提供了多种便捷访问远程Http服务的方法
    return new RestTemplate();
}
```

control更改，这时，直接使用服务端**spring.application.name= provider-dept8001**来访问

```java
// private final String url = "http://localhost:8001";
 private final String url = "http://provider-dept8001";


 @GetMapping("getUser")
 public List getUser(){
     return restTemplate.getForObject(url+"/getUser",List.class);
 }
```

## Ribbon的负载均衡

复制一个与dept-8001相同的项目，该端口为8002，并且配置文件instance-id需要更改，其他不更改，运行80端口地址，发现会轮询的访问这两个dept服务，如果有一个服务挂了，也会去访问，ribbon通过spring.application.name来访问服务端，如果application系统，就会通过负载均衡的方式来访问

```yaml
eureka:
  instance:
    instance-id: provider-dept-8002
```

## Ribbon的内置负载均衡

他们都是IRule接口的实现类

| 策略名                    | 策略描述                                                     | 实现说明                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BestAvailableRule         | 选择一个最小的并发请求的server                               | 逐个考察Server，如果Server被tripped了，则忽略，在选择其中ActiveRequestsCount最小的server |
| AvailabilityFilteringRule | 过滤掉那些因为一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的的后端server（active connections 超过配置的阈值） | 使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就就是检查status里记录的各个server的运行状态 |
| WeightedResponseTimeRule  | 根据相应时间分配一个weight，相应时间越长，weight越小，被选中的可能性越低。 | 一个后台线程定期的从status里面读取评价响应时间，为每个server计算一个weight。Weight的计算也比较简单responsetime 减去每个server自己平均的responsetime是server的权重。当刚开始运行，没有形成statas时，使用roubine策略选择server。 |
| RetryRule                 | 对选定的负载均衡策略机上重试机制。                           | 在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server |
| RoundRobinRule            | roundRobin方式轮询选择server                                 | 轮询index，选择index对应位置的server                         |
| RandomRule                | 随机选择一个server                                           | 在index上随机，选择index对应位置的server                     |
| ZoneAvoidanceRule         | 复合判断server所在区域的性能和server的可用性选择server       | 使用ZoneAvoidancePredicate和AvailabilityPredicate来判断是否选择某个server，前一个判断判定一个zone的运行性能是否可用，剔除不可用的zone（的所有server），AvailabilityPredicate用于过滤掉连接数过多的Server。 |

默认是RoundRobinRule（轮询选择），若想使用其他，为RestTemplate赋予负载均衡功能

```java
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        //RestTemplate提供了多种便捷访问远程Http服务的方法
        return new RestTemplate();
    }
}
```

## 自定义负载均衡

**自定义的配置类不能放在@ComponentScan所扫描的当前包下**

```java
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule(){
        //定义随机算法
        return  new RandomRule();
    }
}
```

```java
@RibbonClient(name = "cloud-zookeeper-comsumer", configuration = MySelfRule.class)
public class PaymentMain80 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain80.class, args);
    }
}
```

## 轮询算法原理

rest接口第N次请求数%服务器总集群数量=实际调用服务器下标

每次服务重启后rest从1开始计算 

# Feign负载均衡

Feign是Netflix开发的声明式、模板化的HTTP客户端， Feign可以帮助我们更快捷、优雅地调用HTTP API。

在Spring Cloud中，使用Feign非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了



## 简单部署

建立一个apimaven包，专门用于写feign的接口

pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

建立一个接口，之后，comsumer端就可以通过这个接口访问打provide端了

```java
@FeignClient(value = "PROVIDER-DEPT") //指定provide的注册服务
public interface DeptClientService {
    //方法名和value得与服务端提供的相同
    @RequestMapping(value="getUser", method = RequestMethod.GET)
    public TUser getUser();
}
```

新建一个80端口的comsumer端

pom的配置

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
</dependencies>
```

配置文件和以前的comsumer一样

建立control类

```java
@RestController
public class UserController {
    @Autowired //只需要引用feign的接口了
    private DeptClientService deptClientService;
    
    @GetMapping("getUser")
    public TUser getUser(){
        return deptClientService.getUser();
    }
}
```

启动类需要@EnableFeignClients来开启feign

```java
@SpringBootApplication
@EnableFeignClients
@EnableEurekaClient
public class DepFeignApplication80 {
    public static void main(String[] args) {
        SpringApplication.run(DepFeignApplication80.class, args);
    }
}
```

# OpenFeign

## 基本使用

> 直连的方式

1. 引入openfeign的jar包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 编写启动类

```java
@SpringBootApplication
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

}
```

3. 在服务端编写一个模拟的接口（获取用户的接口）

```java
@RestController
@RequestMapping("/user")
@Slf4j
public class UserController {
    @GetMapping("/get")
    public UserVO getUser(Long userId) {
        log.debug("接收到请求：{}", userId);
        return new UserVO("张三", 20);
    }
}
```

4. 编写消费端的openfeign代码

```java
@Component
@FeignClient("http://127.0.0.1:81")
@RequestMapping("/user")
public interface UserFeign {
    @GetMapping("/get")
    UserVO getUser(Long userId);
}
```

> 使用注册中心模式

1. 添加注册相关的jar包（如nacos）

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2. 开启注册的注解

```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class OrderApplication
```

3. 将调用的地方的name改为服务方的服务名（user-81）

```java
@Component
@FeignClient(name = "user-81")
@RequestMapping("/user")
public interface UserFeign 
```

## 超时配置

- 有些接口，可能调用会很长时间（openfeign默认超时1s）
- 修改超时的配置

```yaml
# 设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  # 指的是建立连接所用的时间,适用于网络状态正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  # 指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

## 日志增强

- 定义一个config

```java
@Configuration
public class LevelLog {
    @Bean
    public Logger.Level feignLoggerLevel() {
        // 请求和响应的头信息,请求和响应的正文及元数据
        return Logger.Level.FULL;
    }
}
```

- 配置文件

```yaml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.cloud.xiao.controller.FeignController.getDiscovery: debug
```



## 常见错误

- Method Not Allowed
  - 可能是调用的type出错
  - 可能是返回值问题（如返回string， feign接口是实体对象）
  - 可能是服务方式get，消费方使用post调用

在sun.net.www.protocol.http.HttpURLConnection#getOutputStream中，如果 body有值，则自动转为post请求, 而openfeign默认将参数放入body中

所以，如果是get请求，需要带上**RequestParam**

# Hystrix断路器

- 服务降级：服务器忙，请稍后再试，不让客户端等待，立刻返回一个友好提示
  - 触发条件：程序运行异常，超时
- 服务熔断：服务器达到最大访问，出发服务降级方法，是应对雪崩效用的一种微服务链路保护机制，当某个微服务不健康了，**不再调用该微服务，等待微服务正常，恢复调用**

## 问题（服务雪崩）

在分布式系统中，可能会有数十个微服务依赖，如：a->b->c ,这个时候，c出现了问题，那么，c微服务所积累的压力越来越大，这个时候导致整个系统发生更多的级联故障，进而引起系统崩溃，所谓的“雪崩效应”.

这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

## Hystrix作用

### 通配服务降级

服务降级：（反正这个降级我找了许多资料，没搞懂，应该是服务端接收的压力达到设定的值，就不进入这个服务计算，进入预定的方法， 或者，这个服务挂了，我们取一个将就的结果，先凑合着用）

代码实现

在之前的feign的api中修改

增加一个回调函数的类，这个类注入容器中，重写consumer调用的接口（想想dubbo的本地存根）

```java
@Component
public class ClientFallbackService implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public TUser getUser() {
                TUser tUser = new TUser();
                tUser.setUsername("给你个假数据缓缓");
                return tUser;
            }
        };
    }
}
```

在接口中增加注解

```java
@FeignClient(value = "PROVIDER-DEPT", fallbackFactory = ClientFallbackService.class)
public interface DeptClientService {
    @RequestMapping(value="getUser", method = RequestMethod.GET)
    public TUser getUser();
}
```

需要将comsumer端把feign的hystrix的服务降级打开

```yaml
feign:
  hystrix:
    enabled: true
```

关闭了服务器端的服务器，模拟网络断了

访问<http://127.0.0.1/getUser>返回

```json
{"id":null,"username":"给你个假数据缓缓","datesource":null}
```

- 新版写法





### 局部服务降级

一般服务降级fallback是在客户端调用的，这些方法既可以在consumer端也可以在provide端

pom新增

```xml
<!--  hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

```xml
<!--  HSR1版本 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



```java
@SpringBootApplication
@MapperScan("com.xiao.mapper")
//本服务启动后会自动注册进eureka服务中
@EnableEurekaClient
@EnableCircuitBreaker//开启熔断机制
public class Application8003 {
    public static void main(String[] args) {
        SpringApplication.run(Application8003.class, args);
    }
}
```

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;
    @GetMapping("getUser")
    //毁掉的方法
    @HystrixCommand(fallbackMethod = "getUserCallable",
            //调用超过1000ms，则进入回调方法，缺省貌似5000ms
            commandProperties = {
                    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000")})
    public TUser getUser() throws Exception{
        Thread.sleep(2000);//模拟超时
        return userService.findUser();
    }

    public TUser getUserCallable(){
        TUser tUser = new TUser();
        tUser.setId("1");
        //取用户出现问题或者超时，
        // 从缓存取出可能错误的以前的数据，将就下
        tUser.setDatesource("我来自缓存");
        return tUser;
    }
}
```

### 全局服务降级

此处在80客户端

```java
@RestController
//标注全局的降级方法
@DefaultProperties(defaultFallback="fallbackMethod")
public class FeignController {
    @GetMapping(value = "/consumer/timeOut/{id}")
    @HystrixCommand(commandProperties = {
        @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds", value = "1000")
    })
    public String timeOut(@PathVariable("id") Long id) throws Exception{
        return feignService.timeOut(id);
    }
	//当前类访问
    public String fallbackMethod() throws Exception{
        return "全局服务降级方法";
    }
```

### 服务熔断

为了防止雪崩，hystrix提供了基于断路器的服务熔断机制

当请求发生错误的比例超过一定比例时histrix将打开断路器进入closed状态，这个状态下我们所有针对该服务的请求（以依赖隔离的线程池为隔断单位，同一线程池中的所有hiystrix command都会被熔断）都会默认使用降级策略进行处理。结果一段时间（通常称为睡眠窗口）后，断路器将被置为半打开(half open)状态，这个状态下将对一定数量的请求进行正常处理，同时统计请求成功的数量。这个数量如果能达到指定的比例断路器将自动关闭重新进入打开open状态。这就是hystrix基于断路器的熔断器机制

**保证服务方进程不被长时间占用**



代码实现

从8002的providemaven项目复制一个项目8003

pom新增

```xml
<!--  hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
```xml
<!--  HSR1版本 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



```java
@SpringBootApplication
@MapperScan("com.xiao.mapper")
//本服务启动后会自动注册进eureka服务中
@EnableEurekaClient
@EnableCircuitBreaker//开启熔断机制
public class Application8003 {
    public static void main(String[] args) {
        SpringApplication.run(Application8003.class, args);
    }
}
```

```java
/**
     * 在10秒窗口期中10次请求有6次是请求失败的,断路器将起作用(断路器起作用后，就算是正确的也会调用fallback方法，隔一会后才会正常调用)
     顺序：服务降级-》服务熔断-》服务恢复
     * @param id
     * @return
     */
@HystrixCommand(
    fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),// 是否开启断路器
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),// 请求次数
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),// 时间窗口期/时间范文
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")// 失败率达到多少后跳闸
    }
)
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
    if (id < 0) {
        throw new RuntimeException("*****id不能是负数");
    }
    String serialNumber = IdUtil.simpleUUID();
    return Thread.currentThread().getName() + "\t" + "调用成功,流水号:" + serialNumber;
}

public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
    return "id 不能负数,请稍后重试,o(╥﹏╥)o id:" + id;
}
```

## 服务限流

秒杀等高并发操作，严禁一窝蜂拥挤，排队调用方法

## 服务监控hystrixDashboard

能够监控访问的次数与服务是否健康



# zuul路由网关

 

# Gateway新一代网关

## 三大概念

- 路由(Route)
  - 路由为一组断言与一组过滤器的集合，他是网关的一个基本组件

- 断言(Predicate)
  - 我们匹配的条件，为true就进入相对路由

- 过滤器(Filter)
  - 在pre类型的过滤器可以做参数校验，权限校验、流量监控等，
  - 在post类型可以做响应类容，响应头修改，等作用

## 配置

> 引入jar包

- 新建gateway模块，引入jar包（包括：starter-web、nacos-discovery），注意，如果引入zookeeper-discovery，则需要exclusion掉他的默认的zoukeeperjar包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

> 添加配置文件

- 增加配置文件，gateway也是要注册进入注册中心的的
  - id:路由的ID，没有固定规则但要求唯一，简易配合服务名
  - uri:  匹配后提供服务的路由地址
  - predicates: 断言

```yaml
server:
  port: 80

spring:
  application:
    name: gateway-80
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.131:8848
    gateway:
      routes:
        - id: renren_fast_route
          #匹配后提供服务的路由地址
          #uri: http://localhost:8004         
          ## 对注册中心的renren-fast进行轮询请求
          uri: lb://renren-fast
          predicates:
            - Path=/api/**   
```

> 添加启动类

- 网关的启动类其实只需要enable注册中心即可

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

> 启动后前往nacos注册中心能看到对应的配置

- 服务名即spring-name

![image-20210703144901029](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210703144908.png)

## 断言

>  地址断言

```yaml
routes:
    - id: renren_fast_route
      uri: lb://renren-fast
      predicates:
        - Path=/api/** 
```

## 过滤器

> 过滤器配置

在过滤器中，可以鉴权

```java
@Component
public class AuthorFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //模拟获取token
        String token = exchange.getRequest()
            .getHeaders().getFirst("Authorization");
        //模拟获取用户
        String username = exchange.getRequest()
            .getQueryParams().getFirst("username");
        if(token == null) {
            //认证不通过
            exchange.getResponse()
                .setStatusCode(HttpStatus.MULTI_STATUS);
            return exchange.getResponse().setComplete();
        }
        //通过
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //过滤器执行的顺序，0表示第一个执行
        return 0;
    }
}
```

## 路由

### 重写地址

RewritePath：

相当于访问 http://renren-fast/api,重定向的时候，访问第三方地址为

http://renren-fast/renren-fast

```yaml
routes:
  - id: renren_fast_route
    uri: lb://renren-fast
    predicates:
      - Path=/api/**
    filters:
      - RewritePath=/api/?(?<segment>.*), /renren-fast/$\{segment}
```

## 跨域配置

```java
@Configuration
public class AppCorsConfiguration {
    @Bean
    public CorsWebFilter corsWebFilter() {
        //此处要选择reactive的，因为Gateway是响应式的
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.setAllowCredentials(Boolean.TRUE);
        source.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsWebFilter(source);
    }
}
```

- 如果发现类似的错误，测需要将服务端的跨域配置去掉，只需要在网关配置统一跨域

```
header contains multiple values 'http://localhost:8001, http://localhost:8001', but only one is allowed.
```



# Nacos

- Nacos 支持基于 DNS 和基于 RPC 的服务发现（可以作为springcloud的注册中心）、动态配置服务（可以做配置中心）、动态 DNS 服务
- 服务注册+服务配置=eureka+config
- Nacos支持ap+cp模式，可自由切换

## 安装

```shell
## 启动
sh startup.sh -m standalone

## 如果出现类似错误，可以替换 \r
[root@localhost nacos]# ./bin/startup.sh -m standalone
-bash: ./bin/startup.sh: /bin/bash^M: 坏的解释器: 没有那个文件或目录

## 解决方案
[root@localhost nacos]# sed -i 's/\r$//' ./bin/startup.sh
```

- 启动后访问：http://192.168.1.131:8848/nacos/index.html#/login
- 默认密码 nacos / nacos

## pom包引入

- 引入对于版本的cloud和alibaba

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Hoxton.SR10</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.1.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

## 服务注册

> 将一个服务注册到nacos中

- 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现

- 引入jar

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.spring</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>1.0.2</version>
</dependency>
```

- yml配置相应的配置

```yml
server:
  port: 8001
spring:
  application:
    name: nacos-provider-8001
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.131:8848
## 暴露所有端点m
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- 配置启动项

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication8001 {

    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication8001.class, args);
    }
}
```

- 启动成功后能够在nacos中心的 服务管理-服务列表下 看到这个服务

## 配置中心

> nacos能够做统一的配置中心

- Nacos Client 从 Nacos Server 端获取数据时，调用的是此接口 `ConfigService.getConfig`
- nacos配置文件中心：${prefix} - ${spring.profiles.active} . ${file-extension}
- `group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置
- Nacos Config Starter 实现了 `org.springframework.cloud.bootstrap.config.PropertySourceLocator`接口，并将优先级设置成了最高

1. 导入jar包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2. 在resource下新建两个配置文件：新建配置文件 bootstrap.yml

```yaml
spring:
  application:
    name: mall-member
  cloud:
    nacos:
      config:
        # 配置中心地址
        server-addr: 192.168.1.131:8848
        ## 配置中心的文件类型
        file-extension: yaml
  profiles:
    ## 配置环境
    active: dev
```

### 动态刷新

- 调用接口，动态刷新配置

```java
@RestController
@RefreshScope// 发送post的请求，支持动态刷新
public class ConfigController {

    @Value("${config.info}")
    private String config;
    @GetMapping("/getconfig")
    public String getConfig() {
        return config;
    }
}
```

### 命名空间

- spring.cloud.nacos.config.namespace
- 一般生产每个微服务都是一个命名空间
- 它的配置值是创建命名空间的` 命名空间ID`

### Group

- 一般用于想相同微服务，不同的场景
- 如：某个时间段用特殊的配置
- 同时，我们也可以用他来区分是线上环境还是开发环境

### 数据集

- 一般配置太多，我们不会吧配置放到一个文件中
- 如：mybatis配置是一个配置，redis配置是一个配置文件
- 如果没有把配置对于的选项，则使用默认的配置

```yaml
## 使用data-id=mybatis.yaml的配置spring.cloud.nacos.config.extension-configs[0].data-id=mybatis.yaml
```

## 持久化配置

- 为了保证数据存储的一致性，nacos采用集中式存储的方式来支持集群化部署，目前只支持mysql的存储

1. 前往conf目录，寻找sql脚本,在nacos_config数据库中执行脚本

# Sentinel

- 功能：流量控制、熔断降级、系统负载保护
- Hystrix的升级版

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704154543.png)

## 安装

1. 前往sentinel（https://github.com/alibaba/Sentinel）下载 sentinel-dashboard-1.7.1.jar的安装包
2. 将jar包上传linux，编写一个简单脚本

```shell
#! /bin/bash
start(){
  nohup java -jar sentinel-dashboard-1.8.1.jar  > log.file  2>&1 &
}
start;
```

3. 登录：http://192.168.1.131:8080/#/login 密码 sentinel/sentinel

## 应用注入sentinel管理

> 此处我们将user包注入

1. 引入客户端的jar包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2. 配置sentinel的相关配置

```yaml
    sentinel:
      transport:
        dashboard: 192.168.1.131:8080
        #跟控制台交流的端口,随意指定一个未使用的端口即可
        port: 8719
        #客户端的ip，建议配置上
        clientIp: 192.168.1.107
```

启动后能在界面看到对应的配置，也能看到调用的对应接口

**sentinel是懒加载的**

![image-20210704170325027](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704170325.png)

 ## 流控

进入**簇点链路**菜单，查看对应的接口（只有访问过的接口才会出现在菜单里面）

> 名词解释

![image-20210704172344012](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704172344.png)

- 资源名：唯一名称，默认是请求路径

- QPS:每秒请求数
- 线程数： 进入以后只执行对应线程数的请求，当线程数（处理该接口的线程）达到阈值，则进行限流

为了方便测试，建立两个接口

![image-20210704172312208](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704172312.png)

> 测试QPS限制

在对应的地址处点击**流控**

我们对a接口做QPS限制为1，则快速访问A时，发现1秒只能访问1次，如果过多访问，会抛出**Blocked by Sentinel (flow limiting)**异常

> 高级选项
>
> >流控模式-关联

我们对流控做关联调整

![image-20210704172832212](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704172832.png)

- a接口关联b接口，B达到阈值，则限流A（**qps的配置还是配置A这里**）

> > 流控效果

- 预热（Warm Up）
  - 有时，一个接口，平时没有访问，一瞬间请求爆发
  - 让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。warm up冷启动主要用于启动需要额外开销的场景，例如建立数据库连接等。(如：我们设置qps=100,当流量猛增时，我们不让限制一瞬间是100)

- 排队等待：阈值必须是QPS,一个个匀速排队执行（对应算法：漏桶算法）

## 降级



![image-20210704175421667](C:\Users\lonelyxiao\AppData\Roaming\Typora\typora-user-images\image-20210704175421667.png)

> RT(平均响应时间、毫秒级)

- 当1s内持续进入5个请求，对应时刻的平均响应时间(秒级）均超过阈值（count，以ms为单位)，那么在接下的时间窗口（DegradeRule中的timewindow，以s为单位)之内，对这个方法的调用都会自动地熔断(抛出 DegradeException )
- RT最大4900

如：此时，对接口做出限制2000秒阻塞，限制RT=1000ms, 时间窗口=1s



> 异常比例（秒级）

QPS>=5且这个1秒访问的请求数异常比例（报错的请求）超过阈值，触发降级，时间窗口结束后，关闭熔断降级

> 异常比例（秒级）

QPS>=5且这个1秒访问的请求数异常比例（报错的请求）超过阈值，触发降级，时间窗口结束后，关闭熔断降级

> 异常数（分钟级）

当资源近 1 分钟的异常数超过阈值，触发降级，时间窗口（时间窗口配置应该>60s）结束后，关闭熔断降级

## 热点Key限流

热点key的限流往往需要配合SentinelResource注解来进行配置

参数必须是基本类型或者**String**

> SentinelResource兜底方法在同一个类

1. 定义一个热点key的方法，一个兜底方法
   1. 当热点value所配置的规则超过时，出发handler方法

```java
@SentinelResource(value = "hotKey", blockHandler = "blockHandler")
@GetMapping("/hotKey")
public String hotKey(String p1, String p2) {
    return "hotkey";
}

public String blockHandler(String p1, String p2, BlockException exception) {
    return "block error";
}
```

2. 前往界面配置规则
   1. 标识value这个规则，第0个参数，即p1qps达到1以上后，触发兜底方法

![image-20210704205944501](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704205944.png)

> 兜底方法不在同一个类

1. 定义一个类

```java
public class HandlerClass {
    public static String blockHandler(String p1, String p2, BlockException exception) {
        return "block error";
    }
}
```

2. 调整配置
   1. blockHandlerClass标识器配置的类

```java
@SentinelResource(value = "hotKey", blockHandlerClass = HandlerClass.class, blockHandler = "blockHandler")
@GetMapping("/hotKey")
public String hotKey(String p1, String p2) {
    return "hotkey";
}
```

> 特殊值配置

当我们设置了热点key，但是我们还想key在某个特殊值的时候能达到200的阈值，这时，我们可以配置高级选项

![image-20210704210734138](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210704210734.png)



## SentinelResource强化

**SentinelResource**注解，不单单对热点key生效，前面的流控，降级都可以生效，只要将其value配置在对应资源名处，就不是对接口，而是对对应的配置的兜底value生效

> fallback

- fallback针对的是业务上的异常
- blockHandler针对的是sentinel的配置服务

如：此处注意，异常必须是Throwable，因为exceptionsToTrace默认是这个，如果是其他，需要额外配置

```java
@SentinelResource(value = "fallback", fallback = "handlerFallback")
@RequestMapping("/fallback")
public String fallback(String p1) {
    if(Objects.equals(p1, "1")) {
        throw new IllegalArgumentException("发生了异常");
    }
    return "fallback";
}

public String handlerFallback(String p1, Throwable e) {
    return "异常方法....";
}
```

> 整合Openfeign

当我们的服务方网络断了或者挂了，则可以调用兜底方法进行数据的返回

1. 引入注册jar包，sentinel的jar包
2. 前往配置文件开启配置(**激活sentinel对openfeign的支持**)

```yaml
feign:
  sentinel:
    enabled: true
```

3. 更改openfeign的调用类，**注意这个mapping注解不能用**

```java
@FeignClient(name = "user-81", fallback = UserFeignImpl.class)
//@RequestMapping("/user")
public interface UserFeign {
    @GetMapping("/user/get")
    UserVO getUser(@RequestParam("userId") Long userId);
    
}
```

4. 实现兜底方法,这个兜底方法，必须是spring bean

```java
@Component
public class UserFeignImpl implements UserFeign {
    @Override
    public UserVO getUser(Long userId) {
        return new UserVO("服务器凉了，兜底的方法", 20);
    }
}
```

## 系统规则

对整个系统进行配置

## 持久化

如果不配置持久化，则每次服务重启，我们的配置都会消失

此处，我们将持久化配置到nacos中

1. 引入相关jar包

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

2. 配置nacos信息

```yaml
sentinel:
  datasource:
    ds1:
      nacos:
        server-addr: 192.168.1.131:8848
        data-id: user-sentinel
        group-id: DEFAULT_GROUP
        data-type: json
        rule-type: flow
```

3. 前往nacos配置相关配置
   1. 注意data-id一定要一致

![image-20210705001425585](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210705001425.png)

resource:资源名称;

limitApp:来源应用;

grade:阈值类型，0表示线程数，1表示QPS;

count:单机阈值;

strategy:流控模式, O表示直接，1表示关联，2表示链路;

controlBehavior:流控效果，0表示快速失败，1表示Warm Up，2表示排队等待;

clusterMode:是否集群。

- 配置代码

```json
[
    {
        "resource": "/user/a",
        "limitApp": "default",
        "grade": 1,
        "count": 5,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

4. 启动服务后，刷新接口，能够在sentinel看到配置好的配置

![image-20210705001801653](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210705001801.png)

# 分布式事务

## 概念

多数据源的事务控制

> 1+3组件模型

> > 一个事务id

- Transaction ID（XID）：全局事务id

> > 三个组件

- TC - 事务协调者维护全局和分支事务的状态，驱动全局事务提交或回滚。
- TM - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
- RM - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

> 处理过程

- TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
- XID在微服务调用链路的上下文中传播；
- RM(**相当于数据源**)向TC注册分支事务，将其纳入XID对应全局事务的管辖；
- TM向TC发起针对XID的全局提交或回滚决议；
- TC调度XID下管辖的全部分支事务完成提交或回滚请求；

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200330231443.png)

**相当于：TM发起事务的开启或者回滚/提交，TC驱动RM对每个分支数据源进行回滚/提交**

## Server安装

1. 解压seata-server-1.0.0.tar.gz文件
2. 修改conf下的配置文件file.conf
   1. 该文件用于配置`存储方式`、`透传事务信息的NIO`等信息

> 修改配置文件

1. 修改自定义事务组名称:fsp_tx_group
   1. my_test_tx_group:我们自定义的事务组
   2. fsp_tx_group事务组名称

```shell
service {
  vgroup_mapping.my_test_tx_group = "fsp_tx_group"
  default.grouplist = "127.0.0.1:8091"
  disableGlobalTransaction = false
}
```

2. 修改事务日志存储模式为db（默认时文件存储）、

```shell
store {
  mode = "db"
  file {
    dir = "sessionStore"
  }
## 设置数据库信息
  db {
    datasource = "dbcp"
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "mysql"
    password = "mysql"
  }
}
```

3. 创建seata数据库的表信息

可以在源码的：[seata](https://github.com/seata/seata)/[script](https://github.com/seata/seata/tree/develop/script)/[server](https://github.com/seata/seata/tree/develop/script/server)/**db**/的目录找到对应版本的sql脚本

4. 修改**registry.conf**文件（配置的是注册机制）

```shell
registry {
    # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
    type = "nacos"

    nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
}
```

## 事务使用

> 新建三个测试数据库

- seata_order：存储订单的数据库
- seata_storage:存储库存的数据库
- seata_account: 存储账户信息的数据库

> 新建三个模块，模拟操作订单，并且减库存

![image-20210705141807885](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210705141815.png)

> 每个数据库都必须添加一个脚本

```sql
-- 此脚本必须初始化在你当前的业务数据库中，用于AT 模式XID记录。与server端无关（注：业务数据库）
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```



> 以订单为例说明

1. 引入jar包，每个库都要引入seata 的jar包，和mysql相关的jar包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>1.0.0</version>
</dependency>
```

2. 配置文件

```yaml
spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        #与seata server服务器上的对应
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: 192.168.1.131:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.1.134:3306/seata_order
    username: root
    password: 123456
```

3. 将seata的file.conf、registry.conf复制到classpath下

4. 在service层添加代码

```java
@GlobalTransactional
public void create(TOrder order){
    System.out.println("开始创建订单");
    orderDao.insert(order);
    System.out.println("开始减库存");
    storageService.decrease(order.getProductId(), 1);
    System.out.println("开始减金额");
    accountService.decrease(order.getUserId(), order.getMoney());
    System.out.println("开始修改订单状态");
    orderDao.updateStatus(order.getId());
}
```

访问：<http://127.0.0.1:2001/order/create?userId=1&productId=1&money=10&status=0>，发现事务能够回滚

## 原理

> 概念

- TC:全局协调管理者，可以理解为**seata服务器**，负责全局管控
- TM:谁的方法有事务注解标识，它就是事务发起方
- RM:就是上面三个订单库存账户三个库，相当于事务的参与方

> 执行过程

> > 1阶段，seata拦截业务sql

1. 业务数据更新前，将数据保存到before image中

2. 更新后，保存到after image中，生成行锁

> > 2阶段顺利提交，将before和after删除行锁

因为“业务SQL”在一阶段已经提交至数据库，所以Seata框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。

> > 2阶段回滚

回滚方式便是用“before image”还原业务数据;但在还原前要首先要校验脏写，对比“数据库当前业务数据”和“after image”,如果两份数据完全一致就说明没有脏写，可以还原业务数据，如果不一致就说明有脏写，出现脏写就需要转人工处理。
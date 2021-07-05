# 基础

## 版本

Spring Cloud：Hoxton.SR1

<https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/html/spring-cloud.html>

Spring Boot:2.2.2.RELEASE

<https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/html/spring-cloud.html>

## 热部署

1 在对应的项目中添加依赖

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-devtools</artifactId>
     <scope>runtime</scope>
     <optional>true</optional>
</dependency>
```



2 父工程添加插件

```xml
 <build>
        <plugins>
            <plugin>                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

3 idea中，将setting中的compiler的a/b/c/d统统勾上

4 按ctrl+shilft+alt+/弹出选择registry,将compiler.automake勾上，将actionSystem.assertfocus勾上

5重启idea

## RunDashboard配置

可以让多个服务在仪表盘显示

在E:\code\cloudparent\.idea\workspace.xml下配置

```XML
 <component name="RunDashboard">
  	<option name="configurationTypes">
	    <set>
	        <option value="SpringBootApplicationConfigurationType" />
	    </set>
	</option>
```

# 注册中心

## Eureka注册中心

### 服务搭建

springcloud的**Hoxton.SR1**对应的eureka版本

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

配置文件

```yaml
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

建立启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer7001 {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001.class, args);
    }
}
```

进入<http://127.0.0.1:7001/>服务中心页面

### 服务注册

springcloud的**Hoxton.SR1**对应的eureka客户端版本

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

配置文件

```yaml
eureka:
  client:
    ##是否将自己注册进入EurekaServer
    register-with-eureka: true
    ##是否从EurekaServer抓取已有注册信息，默认为true
    #单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
     defaultZone: http://localhost:7001/eureka
```

主启动类添加client启动配置

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
```

### 获取注册服务的信息

开启discoveryClient

```java
@EnableDiscoveryClient
public class PaymentMain8001 {

    public static void main(String[] args) {
```

获取具体信息

```java
	@Autowired
    private DiscoveryClient discoveryClient;
    @GetMapping(value = "/getDiscovery")
    public void getDiscovery(){
        //获取注册服务信息
        List<String> services = discoveryClient.getServices();
        List<ServiceInstance> instances 
            = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for(ServiceInstance instance : instances){
            //根据实例名获取其对应信息
            System.out.println(instance.getServiceId()+" "+instance.getHost()+" ");
        }
    }
```

## 在Zookeeper注册

加入jar包

```xml
<!--SpringBoot整合Zookeeper客户端, 这里面已经引入咯ribbon的包 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <exclusions>
        <!--先排除自带的zookeeper3.5.3-->
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper对应版本-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.0</version>
</dependency>
```

配置文件

```yaml
server:
  port: 8004
spring:
  application:
    name: cloud-payment-service
  cloud:
    zookeeper:
      connect-string: node1:2181
```

获取注册信息,激活Discovery

```java

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8004 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class, args);
    }
}
```

获取注册信息

```java
 @Autowired
    private DiscoveryClient discoveryClient;
    @GetMapping(value = "/getDiscovery")
    public void getDiscovery(){
        //获取注册服务信息
        List<String> services = discoveryClient.getServices();

        List<ServiceInstance> instances = discoveryClient.getInstances("cloud-payment-service");
        for(ServiceInstance instance : instances){
            //根据实例名获取其对应信息
            System.out.println(instance.getServiceId()+" "+instance.getHost()+" ");
        }
    }
```

### zookeeper的注册信息

```shell
[zk: localhost:2181(CONNECTED) 6] ls /services/cloud-payment-service
[57f194ae-27a4-4fec-9e1b-98e5a4aa34ac]

```

### 调用zookeeper注册接口

产生轮询的restTemplate

```java
@Configuration
public class MainConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestRemplate() {
        return new RestTemplate();
    }
}
```

将80端口的服务注册进入zookeeper

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-zookeeper-comsumer
  cloud:
    zookeeper:
      connect-string: node1:2181
```

调用

```java
private String url="http://cloud-payment-service";
@Autowired
private RestTemplate restTemplate;
@GetMapping(value = "/getDiscovery")
public void getDiscovery(){
    String forObject = restTemplate.getForObject(url + "/pay/getDiscovery", String.class);
    System.out.println(forObject);
}
```

## Consul注册中心

开源的分布式服务发现和配置管理系统，go语言开发

功能：服务发现/健康检测/kv存储/多数据中心/可视化web界面

<https://www.springcloud.cc/spring-cloud-consul.html>

# openFeign

## 基本使用

新建一个cloud-consumer-feign-80项目用来测试openfeign，可以发现，其在8003和8004的服务端接口轮询调用

- 主启动类开启openfeign

```java
@SpringBootApplication
@EnableFeignClients
public class FeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(FeignMain80.class,args);
    }
}
```

- 定义一个接口，来调用服务端的接口

服务端接口

```java
@RequestMapping("/pay")
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;
    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping(value = "/getDiscovery/{id}")
    public String getDiscovery(@PathVariable("id") Long id){
```

消费端的接口调用

```java
@Component
//调用的微服务名称
@FeignClient(value = "cloud-payment-service")
@RequestMapping("/pay")
public interface FeignService {
    //需要调用的接口
    @GetMapping(value = "/getDiscovery/{id}")
    String getDiscovery(@PathVariable("id") Long id);
}
```

- 定义controller调用openfeign的接口

```java
@GetMapping(value = "/consumer/getDiscovery/{id}")
public String getDiscovery(@PathVariable("id") Long id) {
    return feignService.getDiscovery(id);
}
```

## 超时设置

有些接口，可能调用会很长时间（openfeign默认超时1s）

配置文件中配置超时时间

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

# Gateway新一代网关

## 与zull1.x的区别

- zuul1.x是一个基于阻塞的io网关

- gateway支持websocket



## 三大概念

- 路由(Route)
  - 路由为一组断言与一组过滤器的集合，他是网关的一个基本组件

- 断言(Predicate)
  - 我们匹配的条件，为true就进入相对路由

- 过滤器(Filter)
  - 在pre类型的过滤器可以做参数校验，权限校验、流量监控等，
  - 在post类型可以做响应类容，响应头修改，等作用

## 路由配置方式1

引入jar包，**注意不要引入spring boot的web相关的jar包**

```xml
<dependencies>
    <!--gateway-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!--SpringBoot整合Zookeeper客户端-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
        <exclusions>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--添加zookeeper对应版本-->
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.6.0</version>
    </dependency>
</dependencies>
```

增加配置文件，gateway也是要注册进入注册中心的的

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud: ## 注册zookeeper
    zookeeper:
      connect-string: node1:2181
    gateway:
      routes:
        - id: payment_routh #payment_routh    #路由的ID，没有固定规则但要求唯一，简易配合服务名
          uri: http://localhost:8004         #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/getDiscovery/**          #断言，路径相匹配的进行路由
```

启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class, args);
    }
}
```

然后访问<http://127.0.0.1:9527/pay/getDiscovery/1>就能访问到8004的服务了

## 路由配置方式2

使用代码的方式将地址映射到其他地址

```java
@Configuration
public class GateWayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("rout_test", r-> r.path("/csdn").uri("https://blog.csdn.net/"));
        return routes.build();
    }
}
```

访问<http://127.0.0.1:9527/csdn>能进入相应的映射

## 网关实现负载均衡

更改网关两处配置

- 将动态网关打开
- 将路由地址改成微服务配置的

```yaml
spring:
  application:
    name: cloud-gateway
  cloud: ## 注册zookeeper
    zookeeper:
      connect-string: node1:2181
    gateway:
      discovery:
        locator:
          enabled: true   #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_routh    #路由的ID，没有固定规则但要求唯一，简易配合服务名
          #uri: http://localhost:8004         #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service  #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/getDiscovery/**          #断言，路径相匹配的进行路由
```

## predicates 断言

- 匹配在after日期之后的请求（在这个时间之后，这个地址才有效）

```yaml
routes:
  - id: payment_routh #payment_routh    #路由的ID，没有固定规则但要求唯一，简易配合服务名
    #uri: http://localhost:8004         #匹配后提供服务的路由地址
    uri: lb://cloud-payment-service  #匹配后提供服务的路由地址
    predicates:
      - Path=/pay/getDiscovery/**          #断言，路径相匹配的进行路由
      - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

这个时间可以这样获取

```java
public static void main(String[] args) {
    ZonedDateTime now = ZonedDateTime.now();
    System.out.println(now);
}
```

- 带cookie的访问， 只有mycookie=mycookievalue时，才能匹配断言，mycookievalue可以是正则表达

```yaml
predicates:
  - Path=/pay/getDiscovery/**          #断言，路径相匹配的进行路由
  - After=2017-01-20T17:42:47.789-07:00[America/Denver]
  - Cookie=mycookie,mycookievalue
```

- 方法

```yaml
predicates:
  - Path=/pay/getDiscovery/**          #断言，路径相匹配的进行路由
  - After=2017-01-20T17:42:47.789-07:00[America/Denver]
  - Cookie=mycookie,mycookievalue
  - Method=GET,POST
```

## 过滤器

当请求参数不满足条件时，不通过请求

```java
@Component
public class CustomGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //过滤器
        String username = exchange.getRequest().getQueryParams().getFirst("username");
        System.out.println(exchange.getRequest().getQueryParams().get("userid"));
        if(username==null){
            //设置状态吗
             exchange.getResponse().setStatusCode(HttpStatus.NOT_IMPLEMENTED);
            //不通过请求
            return exchange.getResponse().setComplete();
        }
        //通过过滤器
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //过滤器的顺序
        return 0;
    }
}
```

# Config分布式配置中心

为所有微服务配置提供一个中心化的配置，将配置信息已REST的方式进行暴露

此处引入zookeeper是将其作为注册中心

## server构建

引入pom

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <!--SpringBoot整合Zookeeper客户端-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
        <exclusions>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--添加zookeeper对应版本-->
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.6.0</version>
    </dependency>
</dependencies>
```

配置

```yaml
server:
  port: 3344


spring:
  application:
    name: cloud-config-center
  cloud:
    zookeeper:
          connect-string: node1:2181
    config:
      server:
        git: #gitee仓库上面的git仓库名字
          uri: https://gitee.com/aloneDr/springcloud-stu-config.git
          ##搜索目录
          search-paths:
            - springcloud-stu-config
      #读取分支
      label: master
```

主启动类

```java
@SpringBootApplication
@EnableConfigServer//启动服务端
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

## 读取地址方式

- <http://127.0.0.1:3344/master/application-dev.yml>
  - 地址/分支名/application-profile.yml

- <http://127.0.0.1:3344/application-dev.yml>
  - 默认读取配置的label

## client构建

引入pom

```xml
<dependencies>
    <!--spring boot 相关-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!--引入客户端的包-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <!--SpringBoot整合Zookeeper客户端-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
        <exclusions>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--添加zookeeper对应版本-->
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.6.0</version>
    </dependency>
</dependencies>
```

建立bootstrap.yml文件

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    zookeeper:
      connect-string: node1:2181
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      #读取后缀名称 上述3个综合：master分支上config-dev.yml的配置文件被读取 http://127.0.0.1:3344/master/config-dev.yml
      profile: dev
      uri: http://127.0.0.1:3344 #配置中心地址
```

启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}
```

获取git上配置信息

```java
@RestController
public class TestController {
    @Value("${config.info}")
    private String value;

    @GetMapping("/getConfig")
    public String getConfig(){
        return this.value;
    }
}
```

访问<http://127.0.0.1:3355/getConfig>获取配置信息

# Stream

屏蔽消息中间件的使用差异，降低切换成本，统一消息编程模型

比如两种消息中间件的数据之间的迁移

遵循的订阅发布模式

## 流程图

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200322210442.jpg)

## 生产者

引入jar

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

定义yml

```yaml
spring:
  application:
    name: cloud-stream-provider
  cloud:
    zookeeper:
      connect-string: node1:2181
    stream:
      binders: # 在此处配置要绑定的rabbitMQ的服务信息
        defaultRabbit: # 表示定义的名称，用于binding的整合
          type: rabbit # 消息中间件类型
          environment: # 设置rabbitMQ的相关环境配置
            spring:
              rabbitmq:
                host: 192.168.94.134
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设为text/plain
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

定义输出的消息发送

```java
//生产者绑定Source
@EnableBinding(Source.class)
public class MessageServiceImpl implements MessageService {

    @Autowired
    private MessageChannel output;

    @Override
    public String send() {
        String s = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(s).build());
        System.out.println("发送一条消息："+s);
        return null;
    }
}
```

消费者

yml

```yaml
spring:
  application:
    name: cloud-stream-provider
  cloud:
    zookeeper:
      connect-string: node1:2181
    stream:
      binders: # 在此处配置要绑定的rabbitMQ的服务信息
        defaultRabbit: # 表示定义的名称，用于binding的整合
          type: rabbit # 消息中间件类型
          environment: # 设置rabbitMQ的相关环境配置
            spring:
              rabbitmq:
                host: 192.168.94.134
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设为text/plain
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

定义订阅者

```java
@EnableBinding(Sink.class)
public class ReceiveMessageListener {
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("消费者1号，------->接收到的消息： "+message.getPayload()+"\t port: "+serverPort);
    }
}
```

## 重复消费

利用steam中的分组

不同组是可以全面消费的（重复消费）

同一组内会发生竞争关系，只有其中一个可以消费

- 配置group
  - 能解决重复消费问题
  - 消息持久丢失问题

```yaml
bindings: # 服务的整合处理
input: # 这个名字是一个通道的名称
destination: studyExchange # 表示要使用的exchange名称定义
content-type: application/json # 设置消息类型，本次为json，文本则设为text/plain
binder: defaultRabbit # 设置要绑定的消息服务的具体设置
group: atguiguA
```

# sleuth

提供了微服务链路服务跟踪方案

## zipkin安装

一条链路通过trace id 为唯一标识，span标识发起的请求信息， 各个span通过parent id关联起来

trace:span的集合，一整条调用的链路存在的唯一标识

span：调用链路的来源，标识一次请求

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200324211701.jpg)

一次调用链不同请求之间的服务依赖关系

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200324212836.png)

- 下载zipKin

<https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/>下载对应版本**zipkin-server-2.12.9-exec.jar**

- 运行zipkin

D:\softinstall>java -jar zipkin-server-2.12.9-exec.jar

- 访问

<http://127.0.0.1:9411/zipkin/>

## sleuth调用监控

以80调用8004为例

- 引入依赖

```xml
<dependencies>
        <!--包含了sleuth+zipkin-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

两个服务都配置

```yaml
spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://127.0.0.1:9411
  sleuth:
    sampler:
      # 采样值，介于0-1之间，一般推荐0.5，1标识全部采样
      probability: 1
```

访问zipkin，查询监控

# Spring Cloud Alibaba

## Nacos

服务注册+服务配置=eureka+config

**Nacos支持ap+cp模式，可自由切换**

### 安装

下载nacos-server-1.1.4.tar.gz，到linux解压，前往bin目录启动（单机模式）

```shell
[root@node1 nacos]# ./bin/startup.sh -m standalone
```

访问

<http://192.168.94.131:8848/nacos/index.html#/login>

nacos/nacos

### 注册

将一个服务注册进入nacos

引入jar

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

配置文件

```yaml
server:
  port: 8081
spring:
  application:
    name: nacos-provider-payment
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.94.131:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosMain9001 {

    public static void main(String[] args) {
        SpringApplication.run(NacosMain9001.class, args);
    }
}
```

进入nacos服务中心，能看到服务管理-服务列表下有这个服务

### Config（统一配置）

- 引入jar包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

- 新建两个配置文件 bootstrap.yml、application.yml

application.yml配置对应的开发环境

```yaml
spring:
  profiles:
    active: dev #开发环境
```

- bootstrap.yml配置通用配置

在nacos配置的文件名规则

${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file.extension}

```yaml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.94.131:8848 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.94.131:8848 #Nacos作为配置中心地址
        file-extension: yaml  #指定yaml格式的配置

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file.extension}
# 配置的nacos的文件名：nacos-config-client-dev.yaml
```

- 登录nacos，在配置列表新增Data Id：nacos-config-client-dev.yaml文件

```yaml
config: 
    info: 'form nacos'
```

- 建立访问类

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

### 分类配置

这个配置会读取nacos上的TEST_GROUP组的配置，命名空间也是同样配置

```yaml
cloud:
  nacos:
    discovery:
      server-addr: 192.168.94.131:8848 #Nacos服务注册中心地址
    config:
      server-addr: 192.168.94.131:8848 #Nacos作为配置中心地址
      file-extension: yaml #指定yaml格式的配置
      group:  TEST_GROUP
```

### 集群和持久化配置

为了保证数据存储的一致性，nacos采用集中式存储的方式来支持集群化部署，目前只支持mysql的存储

- 前往conf目录，寻找sql脚本,在nacos_config数据库中执行脚本

```shell
[root@node1 conf]# pwd
/home/nacos/conf
[root@node1 conf]# vim nacos-mysql.sql 
```

- 备份properties的文件

```shell
[root@node1 conf]# cp application.properties application.properties.init
```

- 配置mysql库

```mysql
###############在此处新增配置
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://192.168.94.134:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

- 配置集群文件

```shell
[root@node1 conf]# cp cluster.conf.example cluster.conf
```

ip必须是

```shell
[root@node1 conf]# hostname -i
192.168.94.131
```

```conf
192.168.94.131:3333
192.168.94.131:4444
192.168.94.131:5555
```

- 修改startup.sh启动文件,表示可以自定义启动的端口

```shell
# 备份文件
[root@node1 bin]# cp startup.sh startup.sh.bak
```

- 修改前

```shell
while getopts ":m:f:s:" opt
do
    case $opt in
        m)
            MODE=$OPTARG;;
        f)
            FUNCTION_MODE=$OPTARG;;
        s)
            SERVER=$OPTARG;;
        ?)
        echo "Unknown parameter"
        exit 1;;
    esac
done
```

```shell
##表示传参数p
while getopts ":m:f:s:p:" opt
do
    case $opt in
        m)
            MODE=$OPTARG;;
        f)
            FUNCTION_MODE=$OPTARG;;
        s)
            SERVER=$OPTARG;;
        p)  
        	PORT=$OPTARG;;
        ?)
        echo "Unknown parameter"
        exit 1;;
    esac
done

```

修改前

```shell
# start
echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
echo "nacos is starting，you can check the ${BASE_DIR}/logs/start.out"
```

修改后

```shell
# start
echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
echo "nacos is starting，you can check the ${BASE_DIR}/logs/start.out"

```

- 启动脚本

```shell
[root@node1 bin]# ./startup.sh -p 3333
[root@node1 bin]# ./startup.sh -p 4444
[root@node1 bin]# ./startup.sh -p 5555
```

- 配置nginx

```conf
 upstream cluster {
          server 192.168.94.131:3333;
          server 192.168.94.131:4444;
          server 192.168.94.131:5555;
        }


        server {
            #监听的端口,这里为80
                listen       80;
                #server_name就是域名,
                server_name  localhost;

                #location域名代理地址
            # / 代表所有请求路径
                location / {
                        #root /home/nginx/html;
                        #index  index.html;
                        proxy_pass http://cluster;
                }
        }

```



- 访问<http://192.168.94.134/nacos/index.html

## Sentinel

前往sentinel（https://github.com/alibaba/Sentinel）下载 sentinel-dashboard-1.7.1.jar

Hystrix的升级版

### Sentinel控制台

- 安装容错、限流控制台

```shell
D:\study\soft>java -jar sentinel-dashboard-1.7.1.jar
```

访问<http://127.0.0.1:8080/#/login>  sentinel/sentinel

### 建立测试包

- 引入jar

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

- 配置

```yaml
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.94.131:8848
    sentinel:
      transport:
        # sentinel dashboard地址
        dashboard: 127.0.0.1:8080
        port: 8719


management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- 主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class SentinelMain8401 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelMain8401.class, args);
    }
}
```

- 测试controller，建立两个访问的地址

```java
@RestController
public class FlowLimitController {

    @GetMapping("getA")
    public String getA() {
        return "A";
    }

    @GetMapping("getB")
    public String getB() {
        return "B";
    }
}
```

### 监控

由于sentinel是懒加载的，我们访问一个地址，可以在sentinel界面看到对应的记录

### 流控模式

- 资源名：唯一名称，默认是请求路径
  - QPS:每秒请求数
  - 线程数： 进入以后只执行对应线程数的请求，当线程数达到阈值，则进行限流

在簇点链路里，在对应的地址处点击新增流控,在流量规则中可以编辑和删除

我们对testA做QPS限制为1，则快速访问A时，发现1秒只能访问

- 流控关联
  - A与B关联，B达到阈值，则限流A

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200328123634.png)

- 流控效果
  - 预热（Warm Up）：有时，一个接口，平时没有访问，一瞬间请求爆发
  - 排队等待：阈值必须是QPS,一个个匀速排队执行（对应算法：漏桶算法）

### 降级

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200328130846.png)

- RT(平均响应时间、毫秒级)
  - 平均响应时间 超出阈值且这些超过阈值的请求数>=5, 则触发熔断降级
  - 窗口期过后，关闭断路器
  - RT最大4900
- 异常比例（秒级）
  - QPS>=5且这个1秒访问的请求数异常比例（报错的请求）超过阈值，触发降级，时间窗口结束后，关闭熔断降级
- 异常数（分钟级）
  - 当资源近 1 分钟的异常数超过阈值，触发降级，时间窗口（时间窗口配置应该>60s）结束后，关闭熔断降级

### 热点Key限流

- 基本配置

对该资源，每秒传入的第N个参数超过阈值

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200329120459.png)

- 参数例外项

当第N个参数是某个例外值时，可以不受限制，或者限制更改

### 系统规则

对整个系统进行配置

### @SentinelResource 注解

- 当getc触发限流，则进入blockHandler指定的方法

```java
@GetMapping("getc")
@SentinelResource(value = "getc", blockHandler = "blockHandlerc")
public String getc(){
    return "getc";
}

public String blockHandlerc(BlockException exception) {
    //打印执行的方法名
    System.out.println(exception.getClass().getCanonicalName());
    return "exception blockc";
}
```

- 降低代码的耦合度，兜底方法写到另一个类中
  - 建立统一的兜底方法
  - 引入兜底方法

```java
public class GetDHandler {
    public static String blockHandlerD(BlockException excepion) {
        return "blockHandlerD";
    }
}
```

```java
@GetMapping("/getD")
@SentinelResource(value = "getD",
                  blockHandlerClass = GetDHandler.class,//去GetDHandler类
                  blockHandler = "blockHandlerD")//寻找blockHandlerD方法
public String getD() {
    return "getD";
}
```

### 基于openFeign熔断

- 

pom

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
</dependencies>
```

配置文件

```yaml
server:
  port: 80

spring:
  application:
    name: nacos-openfeign-client
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.94.131:8848 #Nacos服务注册中心地址
# 激活sentinel对openfeign的支持
feign:
  sentinel:
    enabled: true
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class CloudAlibabaOpenFeign80 {
    public static void main(String[] args) {
        SpringApplication.run(CloudAlibabaOpenFeign80.class, args);
    }
}
```

配置访问的service接口 **注意这个mapping注解不能用**

```java
@FeignClient(value = "nacos-provider-payment", fallback = OpenFeignServiceImpl.class)
//@RequestMapping("/pay")
public interface OpenFeignService {
    @GetMapping(value = "/pay/getDiscovery/{id}")
    String getDiscovery(@PathVariable("id") Long id);
}
```

兜底方法，当provide服务断了，则进入这个方法

```java
@Component
public class OpenFeignServiceImpl implements OpenFeignService {
    @Override
    public String getDiscovery(Long id) {
        return "服务端断了,服务降级，openfeign兜底方法";
    }
}
```

调用

```java
@RestController
public class OpenFeignController {
    @Autowired
    private OpenFeignService openFeignService;
    @GetMapping("/getOpenFeign/{id}")
    public String getOpenFeign(@PathVariable("id") Long id){
        System.out.println("进入openFeign--"+id);
        return openFeignService.getDiscovery(id);
    }
}
```

- 持久化
  - 问题：当调用的controller服务停了后，sentinel的配置会消失

一般持久进入nacos

## seata处理分布式事务

1+3组件模型

Transaction ID xid ：全局事务id

- 三组件概念
  - TC - 事务协调者维护全局和分支事务的状态，驱动全局事务提交或回滚。
  - TM - 事务管理器：定义全局事务的范围：开启全局事务、提交或回滚全局事务。
  - RM - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

### 处理过程

- TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
- XID在微服务调用链路的上下文中传播；
- RM(**相当于数据源**)向TC注册分支事务，将其纳入XID对应全局事务的管辖；
- TM向TC发起针对XID的全局提交或回滚决议；
- TC调度XID下管辖的全部分支事务完成提交或回滚请求；

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/SpringCloud/20200330231443.png)

### 安装

- 修改**file.conf**文件

该文件用于配置`存储方式`、`透传事务信息的NIO`等信息

​	修改自定义事务组名称:fsp_tx_group

```conf
service {
  #transaction service group mapping
  vgroup_mapping.my_test_tx_group = "fsp_tx_group"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  #disable seata
  disableGlobalTransaction = false
}
```

修改事务日志存储模式为db

```conf
store {
  ## store mode: file、db
  mode = "db"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "mysql"
    password = "mysql"
  }
}
```

- 创建seata数据库

[seata](https://github.com/seata/seata)/[script](https://github.com/seata/seata/tree/develop/script)/[server](https://github.com/seata/seata/tree/develop/script/server)/[db](https://github.com/seata/seata/tree/develop/script/server/db)/**mysql.sql**

- 修改**registry.conf**文件

```conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "file"

  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
```

### 业务数据库准备

- 新建三个数据库
  - seata_order：存储订单的数据库
  - seata_storage:存储库存的数据库
  - seata_account: 存储账户信息的数据库
- 创建表

```sql
create table t_order(
	id bigint(11) auto_increment,
	user_id bigint comment '用户id',
	product_id bigint comment '产品id',
	money int comment '金额（分）',
	status int(1) comment '0 创建中 1 完结中',
	primary key(id)
) comment '订单表';
```

```sql
create table t_storage(
	id bigint(11) auto_increment,
	product_id bigint comment '产品id',
	total int(11) comment '数量',
	reside int(11) comment '剩余库存',
	primary key(id)
) comment '库存表';	
```

```sql
create table t_account(
	id bigint(11) auto_increment,
	user_id bigint(11) comment '用户id',
	total int(11) comment '总额度',
	reside int(11) comment '剩余额度',
	primary key(id)
) comment '账户表';
```

- 在每个数据库建立日志回滚表

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

### 新建三个模块

- 业务需求

下订单->减库存->扣余额->改订单状态

- 新建三个模块
  - 订单
  - 库存
  - 账户

- 以订单模块为例

pom

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

配置文件

```yaml
server:
  port: 2001
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
        server-addr: 192.168.94.131:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.94.134:3306/seata_order
    username: root
    password: 123456

mybatis:
  # 给实体类的包目录起别名
  type-aliases-package: com.xiao.seata.Entity
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

将seata的file.conf、registry.conf复制到classpath下

file.conf需要修改

```conf
service {
   #修改自定义事务组名称
  vgroup_mapping.fsp_tx_group = "default"
  default.grouplist = "192.168.94.131:8091"
  disableGlobalTransaction = false
}
```

整合数据源，使用seata的数据源配置

```java
@Configuration
public class DataSourceProxyConfig {
    @Value("${mybatis.mapper-locations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }

    @Primary
    @Bean("dataSource")
    public DataSourceProxy dataSourceProxy(DataSource druidDataSource) {
        return new DataSourceProxy(druidDataSource);
    }

    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSourceProxy);
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        bean.setMapperLocations(resolver.getResources(mapperLocations));
        bean.setTransactionFactory(new SpringManagedTransactionFactory());
        return bean.getObject();
    }
}
```

在service层的方法加上对应的事务注解

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

### 原理

- 概念
  - TC:全局协调管理者，可以理解为seata服务器，负责全局管控
  - TM:谁的方法有事务注解标识，它就是事务发起方
  - RM:就是上面三个订单库存账户三个库，相当于事务的参与方

- 执行过程

  - 1阶段，seata拦截业务sql

  业务数据更新前，将数据保存到before image中

  更新后，保存到after image中，生成行锁

  ```
  观察业务数据表，发现执行到orderDao.insert(order);时，t_order有了一条数据，undo_log也多了一条数据，这个表存的是前后镜像数据
  ```

  

  数据放入各个业务表的undo_log中

  - 2阶段顺利提交，将before和after删除行锁
  - 2阶段回滚，那么after和业务数据库对比，一至，将前置数据逆向数据回滚

  

### seata库说明

- branch_table：当一个rm执行sql后，会往这个表插入一条分支数据
- global_table： 记录事务发起方信息，并且记录全局信息
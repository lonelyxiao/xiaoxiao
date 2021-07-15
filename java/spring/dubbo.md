# Spring Boot与Dubbo

## 基础配置

- pom引入

```xml
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>${nacos.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>${dubbo.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo</artifactId>
    <version>${dubbo.version}</version>
</dependency>
```

> 服务器端

- 需要配置暴露的协议和端口（-1表示随机）
- 需要配置扫描的service的目录

```yml
nacos:
  server-address: 127.0.0.1
  port: 8848
  username: nacos
  password: nacos

dubbo:
  scan:
    base-packages: com.xiao.manager
  registry:
    address: nacos://${nacos.server-address}:${nacos.port}/?username=${nacos.username}&password=${nacos.password}
  protocol:
    port: -1
    name: dubbo
```

服务端的暴露的service

```java
@DubboService(version = "1.0.0")
@Slf4j
public class DemoManagerImpl implements DemoManager {
    @Override
    public String sayHello(String name) {
        log.debug("来自{} 的请求...{}", LocalDateTime.now());
        return "收到:"+name;
    }
}
```

> 消费端

```yaml
nacos:
  host: 127.0.0.1
  port: 8848
  username: nacos
  password: nacos

dubbo:
  registry:
    address: nacos://${nacos.host}:${nacos.port}/?username=${nacos.username}&password=${nacos.password}
```

```java
@DubboReference(version = "1.0.0")
public DemoManager demoManager;


@GetMapping("/sayHello/{name}")
public String sayHello(@PathVariable("name") String name) {
    return demoManager.sayHello(name);
}
```

## 超时配置

消费端：reference默认使用的超时为 consumer的timeout:1000

​				可也以如此配置@Reference(timeout = 6000)

也可在服务端设置timeout

方法级别>类级别

消费端>服务端

# 集群容错

> 重试次数

Dubbo 服务在尝试调用一次之后，如出现非业务异常(服务突然不可用、超时等)，Dubbo 默认会进行额外的最多2次重试.（也就是3次）

重试次数支持两种自定义配置: 1.通过注解/xml进行固定配置；2.通过上下文进行运行时动态配置

在**服务端**的注入注解配置重试次数（消费端也可以配偶之，一般服务器端配置）

```java
@DubboService(retries = 1)
```

> 容错模式

- Failover Cluster

失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含第一次)。

- Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

- Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

- Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

# 服务降级

``当服务器压力剧增的情况下根据实际业务情况及流量，对士些服务和页面有策略的不处理或换和简单的方式处理,从而释放服务器资源以保证核心交易正常运作或高效运作。``

需要在admin的控制台配置

> 屏蔽

mock=force:return+null表示消费方对该服务的方法调用都直接返回null值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。

![image-20210710125758954](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210710125806.png)

> 容错

还可以改为mock=fail:return+null表示消费方对该服务的方法调用在失败后，再返回 null值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。（如**超时**等操作）

![image-20210710125828802](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210710125828.png)

# 整合Hystrix断路器

# dubbo原理

## 整体设计

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210710152556.jpg)

1. service ： 暴露给用户调用的一层，通过接口，调用远程的方法
2. config: 配置层，配置层收集配置信息
3. proxy: 代理层，通过代理的方式，生成客户端的代理对象
4. registry：注册中心层，生产者服务注册进入注册中心，消费者从注册中心发现注册服务
5. cluster:路由层，调用者通过路由的算法负载均衡的方式调用消费者
6. monitor:监控层，每一次调用都能在监控中心看到
7. protoco: 远程调用层，封装整个RPC调用
8. exchange：通信层，架起NIO的通信管道
9. transport： 传输层，通过transporter传输
10. serialize: 序列化层

## 初始化流程

> @EnableDubbo,会import DubboConfigConfigurationRegistrar的bean，这个bean有个bind的注解

```java
@EnableConfigurationBeanBindings({
        @EnableConfigurationBeanBinding(prefix = "dubbo.application", type = ApplicationConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.module", type = ModuleConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.registry", type = RegistryConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.protocol", type = ProtocolConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.monitor", type = MonitorConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.provider", type = ProviderConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.consumer", type = ConsumerConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.config-center", type = ConfigCenterBean.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.metadata-report", type = MetadataReportConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.metrics", type = MetricsConfig.class),
        @EnableConfigurationBeanBinding(prefix = "dubbo.ssl", type = SslConfig.class)
})
```

EnableConfigurationBeanBinding会import这些注解上的bean,然后将配置信息封装到这些属性里面

> @DubboComponentScan注解import了DubboComponentScanRegistrar

1. 这个注解注入了ServiceAnnotationBeanPostProcessor、ReferenceAnnotationBeanPostProcessor等BeanDefinitionRegistryPostProcessor的实现类
2. 可以在bean定义结束后，再新增一些bean的定义

> >  BeanDefinitionRegistryPostProcessor

这个processor将包路径下的dubboservice相关注解，和ServiceBean/DubboBootstrapApplicationListener的bean

## 服务暴露流程

> ~~ServiceBean是服务标签的核心类~~

~~它实现了：~~

~~InitializingBean：在属性设置完以后会回调afterPropertiesSet~~

~~它通过exported方法发布事件来进行暴露服务~~

上面的是老版本

> DubboBootstrapApplicationListener

它实现了ApplicationListener，发布事件onApplicationEvent调用DubboBootstrap#start

1. 在ServiceConfig#doExportUrls中获取暴露的协议，进行循环暴露

```java
for (ProtocolConfig protocolConfig : protocols) {
    doExportUrlsFor1Protocol(protocolConfig, registryURLs);
}
```

2. 在ServiceConfig#doExportUrlsFor1Protocol调用PROTOCOL.export(wrapperInvoker)进行服务暴露
   1. PROTOCOL是通过spi加载的对象，dubbo协议的就是DubboProtocol类对象
3. 先到RegistryProtocol#export
4. 在RegistryProtocol的doLocalExport方法中调用DubboProtocol#export,进行服务的地址的绑定
5. DubboProtocol#export的openServer方法中，在createServer使用Transporters.bind进行netty的服务创建

6. 回到RegistryProtocol#export来进行对应的信息注册进入注册中心

## 服务引用流程

> 服务引用先看ReferenceBean

1. 他是FactoryBean的实现这，在注入bean的时候会调用FactoryBean#getObject方法
2. 在ReferenceConfig#init创建代理对象

```java
ref = createProxy(map);
```

3. 调用invoker = REF_PROTOCOL.refer(interfaceClass, urls.get(0));方法

4. 进入RegistryProtocol#refer方法获取注册中心的信息

```java
//通过这个方法获取注册中心的地址信息
Invoker<T> invoker = cluster.join(directory);
```

# Dubbo执行流程

 start: 启动Spring容器时,自动启动Dubbo的Provider

1、register: Dubbo的Provider在启动后自动会去注册中心注册内容.注册的内容包括:

1. Provider的 IP
2. Provider 的端口.
3. Provider 对外提供的接口列表.哪些方法.哪些接口类

1.4 Dubbo 的版本.

1.5 访问Provider的协议.

2、subscribe: 订阅.当Consumer启动时,自动去Registry获取到所已注册的服务的信息.

3、notify: 通知.当Provider的信息发生变化时, 自动由Registry向Consumer推送通知.

4、invoke: 调用. Consumer 调用Provider中方法

4.1 同步请求.消耗一定性能.但是必须是同步请求,因为需要接收调用方法后的结果.

5、count:次数. 每隔2分钟,provoider和consumer自动向Monitor发送访问次数.Monitor进行统计.

# 注册中心挂了可以继续通信吗

*可以，因为刚开始初始化的时候，消费者会将提供者的地址等信息拉取到本地缓存，所以注册中心挂了可以继续通信*


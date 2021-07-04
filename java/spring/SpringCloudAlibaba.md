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
- 如果没有吧配置对于的选项，则使用默认的配置

```yaml
## 使用data-id=mybatis.yaml的配置
spring.cloud.nacos.config.extension-configs[0].data-id=mybatis.yaml
```

## 持久化配置

- 为了保证数据存储的一致性，nacos采用集中式存储的方式来支持集群化部署，目前只支持mysql的存储

1. 前往conf目录，寻找sql脚本,在nacos_config数据库中执行脚本
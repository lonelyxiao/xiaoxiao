# 基础配置

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

- 服务器端
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

- 消费端

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

# 重试机制


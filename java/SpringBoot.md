**本笔记为观看雷老师视频所书**

**萧冀豪**

# spring boot 入门

目的：快速的搭建spring的产品级的环境

## spring boot 启动器

启动器其实就是jar包的集合

导入父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.8.RELEASE</version>
    <relativePath />
</parent>
```

如：支持全栈的开发，如spring mvc的jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

如：与redis集合的jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

主程序入口，进入注解源码

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {        @Filter(
            type = FilterType.CUSTOM,
            classes = {TypeExcludeFilter.class}
        ),         @Filter(
            type = FilterType.CUSTOM,
            classes = {AutoConfigurationExcludeFilter.class}
        )}
)
public @interface SpringBootApplication
```

### @SpringBootApplication注解

在@SpringBootConfiguration中可以看到

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //相当于一个spring bean 的xml配置文件
public @interface SpringBootConfiguration
```

**@EnableAutoConfiguration**是开启自动配置的注解+扫描对应的包

进入其中看到如下代码

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration
```

**@AutoConfigurationPackage**自动配置包

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage
```

**@Import({Registrar.class})** spring注解，导入组件

导入的组件：

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        //(arg0)registerBeanDefinitions：注册一些bean信息
        //(arg1)拿到要注入的bean的包：如 Application同级别以下的包
        //，将其下的注解放入spring IOC容器中
        AutoConfigurationPackages.register(registry, new String[]{(new AutoConfigurationPackages.PackageImport(metadata)).getPackageName()});
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImport(metadata));
    }
}
```

**@Import({EnableAutoConfigurationImportSelector.class})**注解：导入自动装配的组件

去看源码，将所有的组件导入，将全类名返回

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if(!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        try {
            AutoConfigurationMetadata ex = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //返回的组件方法,获取候选的配置
            List configurations = this.
                getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            configurations = this.sort(configurations, ex);
            Set exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, ex);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return (String[])configurations.toArray(new String[configurations.size()]);
        } catch (IOException var6) {
            throw new IllegalStateException(var6);
        }
    }
}
```

this.getSpringFactoriesLoaderFactoryClass()获取到的是EnableAutoConfiguration.class

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    //这个方法将自动装配组件加载进入容器（从META-INF/spring.factories文件中获取）
    List configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

loadFactoryNames下代码：factoryClass为EnableAutoConfiguration，

从方法中我们可以看到

将META-INF/spring.factories配置的容器的EnableAutoConfiguration的配置加载，以数组方式返回，用这些配置来自动配置，如：org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration容器

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    try {
        Enumeration ex = classLoader != null?classLoader.getResources("META-INF/spring.factories"):ClassLoader.getSystemResources("META-INF/spring.factories");
        ArrayList result = new ArrayList();

        while(ex.hasMoreElements()) {
            URL url = (URL)ex.nextElement();
            Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
        }

        return result;
    } catch (IOException var8) {
        throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + "META-INF/spring.factories" + "]", var8);
    }
}
```

## 自动装配举例

以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；



用springboot开发的过程中，我们会用到**@ConfigurationProperties**注解，主要是用来把properties或者yml配置文件转化为bean来使用的，而**@EnableConfigurationProperties**注解的作用是@ConfigurationProperties注解生效。
如果只配置@ConfigurationProperties注解，在IOC容器中是获取不到properties配置文件转化的bean的，当然在@ConfigurationProperties加入注解的类上加@Component也可以使交于springboot管理。

@EnableConfigurationProperties：如果@ConfigurationProperties想生效得注入容器中，如果没有注解注入，则加上EnableConfigurationProperties就会生效

```java
@Configuration 
@EnableConfigurationProperties({HttpEncodingProperties.class})
@ConditionalOnWebApplication//判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnClass({CharacterEncodingFilter.class})//判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true
) //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
    private final HttpEncodingProperties properties;
    //只有一个有参构造器的情况下，参数的值就会从容器中拿 @EnableConfigurationProperties
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

## 打包

在pom中配置,表示打包成一个可执行的spring boot 的jar包

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## 注意点

App启动类需要在其扫描的包同级或者同级之上

# 配置文件

##  yaml配置文件

```yaml
#自定义注入yaml的属性（com.xiao.bean.TestYaml）
testyaml:
  name: zhangsan
  name2: "zhang \t san"
  name3: 'zhang \t san'
  user1:
    name: zhangsan #对象第一种方式
  user2: {name: zhangsan} #对象第二种方式
  map:
    value: map1 # map第一种方式
  map1: {value: map2}
  lists: #z注意，list的变量定义一定以s结尾
    - a
    - b
```

```java
@Component
@ConfigurationProperties(prefix = "testyaml")//表示将那个前缀属性注入这个bean中
public class TestYaml {
    private String name;
    private String name2;//测试双引号
    private String name3;//测试单引号

    private User user1;
    private User user2;

    private Map map;
    private Map map1;
    private List lists;
    private List list1s;
    public String getName() {
        return name;
    }
```

**另一种写法**

其实就相当于往spring bean 的properties属性里注入值

```java
@Controller
public class TestYaml2 {
    @Value("${testyaml.name}")
    private String name;
    @Value("#{3+4}")
    private Integer age;//测试双引号
    @Value("父名称")
    private String name3;//测试单引号
```

##  加载自定义配置文件

这种方式只能加载properties文件

```
@PropertySource(value = "classpath:testyaml.properties")
@Component
@ConfigurationProperties(prefix = "testyaml")
public class TestYaml
```

在classpath下建立testyaml.properties文件

```properties
testyaml.name=zhangsan
testyaml.name2="zhang \t san"
testyaml.name3='zhang \t san'
testyaml.user1.name=haha
testyaml.map.k1=ll
testyaml.lists=1,2,3
```

## 加载spring的配置文件

在classpath下建立spring文件，然后在启动类加载配置，就会加载spring文件，产生对应的bean

```java
@ImportResource(locations = "classpath:test.xml")
@SpringBootApplication
public class App2
```

另一种写法，不写spring.xml,@Bean返回的bean id 默认是方法名

```java
@Configuration
public class TestConfig {

    @Bean
    public TestService testService(){
        return new TestService();
    }
}
```

## 配置文件的占位符

```properties
#当占位符不存在，使用:后面的补充值
testyaml.name=${testyaml.test:补充值}zhangsan
#使用随机值
testyaml.name2=${randon.uuid}zhang san
#使用配置值
testyaml.name3=${testyaml.name}zhang san

testyaml.user1.name=haha
testyaml.map.k1=ll
testyaml.lists=1,2,3
```

## 生产与开发时，配置文件的切换

**使用文件的方式**

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

现在，我们由application.yml、application-dev.yml（开发）、application-prod.yml（生产）三个文件

只需要在application.yml中配置，便可指定对应的配置文件，如指定开发环境

```yaml
spring:
  profiles:
    active: dev 
```

**使用yaml的document方式**

yaml的---表示文档分割线

```yaml
spring:
  profiles:
    active: prod
server:
  port: 8081 #配置端口号


---
server:
  port: 8082 #配置端口号
spring:
  profiles: dev
---
server:
  port: 8083 #配置端口号
spring:
  profiles: prod
```

在激活profile的模式时

**可以在配置文件中**

```yaml
spring:
  profiles:
    active: prod
```

**也可以在jar包运行时**

java -jar xx.jar --spring.profiles.active=dev；

**也可以配置虚拟机参数**

-Dspring.profiles.active=dev

## 配置文件的优先级

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件（file表示项目根目录）

–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**（高优先级的配置会覆盖低优先级的相同配置属性）；

==我们还可以通过spring.config.location来改变默认的配置文件位置==

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

java -jar xx.jar --spring.config.location=G:/application.properties

# 日志

SpringBoot：底层是Spring框架，Spring框架默认是用JCL；‘

​	**SpringBoot选用 SLF4j和logback**

所以，在springboot使用了 slf4j的中间包来解决兼容问题（具体可查官网替换原则）


日志级别由低到高trace、debug、info、warn、error

springboot默认使用info级别

```java
package com.xiao.log;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * Created by Administrator on 2019/5/20.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestLog {
    Logger logger = LoggerFactory.getLogger(this.getClass());

    @Test
    public void test1(){

        logger.trace("---trace---");
        logger.debug("----debug---");
        logger.info("----info---");
        logger.warn("----warn");
        logger.error("---error");
    }
}
```

日志输出配置

   日志输出格式：
		%d表示日期时间，
		%thread表示线程名，
		%-5level：级别从左显示5个字符宽度
		%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
		%msg：日志消息，
		%n是换行符
    %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n

```yaml
logging:
  level: #更改日志级别，level是map类型，指定所在包下的日志级别
    com:
      xiao: trace
  #file: d:/springlog.log ##指定日志输出路径，带日志文件
  path: /spring/log ##在当前磁盘下的目录写入日志
  pattern:
    #日志控制台输出的格式,注意带单引号，否则会转义
    console: '%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n'
    #文件目录中输出的格式
    file: '%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n'
```



### 指定配置文件方式

给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

**logback-spring.xml**：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>

```

如：<springProfile name="!dev">指定是否时开发环境

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```


## 访问静态资源

### 在classes/static下寻找

在目录下放入图片

\src\main\resources\static\java.jpg

通过<http://localhost:8080/java.jpg>可以访问

## 文件上传

### @RestController详解

```
表示这个类的方法类默认返回转为json，不再需要@ResponseBody
```

### 代码

```java
@RestController //表示这个类的方法类默认返回转为json，不再需要@ResponseBody
public class FileController {


    @RequestMapping("/fileupload")
    public Map fileUpload(MultipartFile filename){
        Map map = new HashMap<>();
        map.put("name", filename.getOriginalFilename());
        return map;
    }
}
```

```yml
spring:
  http:
    multipart:
      # 允许的单个文件上传大小
      maxFileSize: 200MB
      ## 允许的总文件上传大小
      maxRequestSize: 200MB
```

# spring boot WEB开发

## 静态资源映射

在org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration中

webjars：以jar包的方式引用资源，他们可以以maven的方式将前端框架引用进来

以webjars方式引用;classpath:/META-INF/resources/webjars/* 文件

所以引入pom

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
```

再访问<http://localhost:8083/webjars/jquery/3.3.1/jquery.js>，就能访问到jquery文件（只需要访问webjars下的resources下路径）

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if(!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Integer cachePeriod = this.resourceProperties.getCachePeriod();
        if(!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(cachePeriod));
        }

        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        if(!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(this.resourceProperties.getStaticLocations()).setCachePeriod(cachePeriod));
        }

    }
}

@Configuration
@ConditionalOnProperty(
    value = {"spring.mvc.favicon.enabled"},//配置图标配置
    matchIfMissing = true
)
public static class FaviconConfiguration {
    private final ResourceProperties resourceProperties;
    
    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(-2147483647);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", this.faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(this.resourceProperties.getFaviconLocations());
        return requestHandler;
    }
```



```java
@ConfigurationProperties(
    prefix = "spring.resources", //资源有关参数配置
    ignoreUnknownFields = false
)
public class ResourceProperties implements ResourceLoaderAware
```



String staticPathPattern = this.mvcProperties.getStaticPathPattern();这个里面的配置

/**， 所以，这是另一个静态资源 ，这表示如果没有webjars的路径，则走这个逻辑，进入

this.resourceProperties.getStaticLocations()中可以看到，他包含的路径为这些，所以这几个路径都可以放静态资源

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
```

---

## 欢迎页的配置

```java
@Bean
public WebMvcAutoConfiguration.WelcomePageHandlerMapping welcomePageHandlerMapping(ResourceProperties resourceProperties) {
    //第一个参数，欢迎页  第二个参数，被谁映射，这里时/**
    return new WebMvcAutoConfiguration.WelcomePageHandlerMapping(resourceProperties.getWelcomePage(), this.mvcProperties.getStaticPathPattern(), null);
}
```

在getWelcomePage看到欢迎页的代码，被/**映射,我们可以在static文件下放入index.html文件，访问<http://localhost:8083/>，发现进入了index.html页面

```java
private String[] getStaticWelcomePageLocations() {
    String[] result = new String[this.staticLocations.length];

    for(int i = 0; i < result.length; ++i) {
        String location = this.staticLocations[i];
        if(!location.endsWith("/")) {
            location = location + "/";
        }

        result[i] = location + "index.html";
    }

    return result;
}
```



## spring boot整合thymeleaf

templates：在resources目录下，与static目录相似，但不能被外界直接访问，必须经过一层视图渲染

## thymeleaf 的包升级

1.5的springboot 默认使用thymeleaf2.0的jar包，这个包缺点就是html需要严格按照html格式，所以需要升级

代码部分（这里的原理及时覆盖parent的里面的定义的版本属性）

index.html：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>第一个视图渲染层</title>
</head>
<body>
    <!--使用常量字符串-->
    <span th:text="hello"></span>
    <!--使用动态的数据-->
    <span th:text="${msg}"></span>
</body>
</html>
```

controller:

```java
@Controller
public class ThymeleafController {

    @RequestMapping("first")
    public String first(Model model){
        model.addAttribute("msg", "哈哈哈哈");

        return "index";
    }
}
```

pom.xml

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
    <!--jar包升级-->
    <thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.0.4</thymeleaf-layout-dialect.version>
</properties>


<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

## thymeleaf使用

在自动配置的类ThymeleafAutoConfiguration中，找到Thymeleaf3Configuration的配置进入ThymeleafProperties类可以看到如下代码，

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");
    private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML5";
```

使用：

1、导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2、使用thymeleaf语法；

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```



## Spring mvc 自动注册

日期格式化的注册，可以看到，如果没配置spring.mvc的配置，则不会格式化

```java
@Bean
@ConditionalOnProperty(
    prefix = "spring.mvc",
    name = {"date-format"}
)
public Formatter<Date> dateFormatter() {
    return new DateFormatter(this.mvcProperties.getDateFormat());
}
```

从这个源码中可以看到，只要我们往bean里面注入实现了converter等接口的bean，则会自动进行注入生效

```java
public void addFormatters(FormatterRegistry registry) {
    Iterator var2 = this.getBeansOfType(Converter.class).iterator();

    while(var2.hasNext()) {
        Converter formatter = (Converter)var2.next();
        registry.addConverter(formatter);
    }

    var2 = this.getBeansOfType(GenericConverter.class).iterator();

    while(var2.hasNext()) {
        GenericConverter formatter1 = (GenericConverter)var2.next();
        registry.addConverter(formatter1);
    }

    var2 = this.getBeansOfType(Formatter.class).iterator();

    while(var2.hasNext()) {
        Formatter formatter2 = (Formatter)var2.next();
        registry.addFormatter(formatter2);
    }

}
```

## spring mvc 拓展

如下 平时，我们的地址访问不需要操作，直接跳转 success页面，则进行如下配置，或者配置过滤器

```xml
<mvc:view-controller path="/hello" view-name="success"/>    
```

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

如果我们使用spring boot来完成以上功能，可以如下,

注意：这个类不能标注@EnableWebMvc

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /hello 请求来到 success
        registry.addViewController("/hello").setViewName("success");
    }
}
```

不能标注@EnableWebMvc原因：我们看到这个注解@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
      WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
      ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

原理：看到EnableWebMvcConfiguration类

```java
@Configuration
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter
```

setConfigurers这个方法将所有WebMvcConfigurer的接口的bean注入，并且添加到config中

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

   private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

   @Autowired(required = false)
   public void setConfigurers(List<WebMvcConfigurer> configurers) {
      if (!CollectionUtils.isEmpty(configurers)) {
         this.configurers.addWebMvcConfigurers(configurers);
      }
   }
```

## 国际化

 ## 错误处理机制

### 返回错误页面

浏览器错误界面会返回404

postman进入访问错误界面会返回404json串

原理：

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class })
// Load before the main WebMvcAutoConfiguration so that the error View is available
@AutoConfigureBefore(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties(ResourceProperties.class)
public class ErrorMvcAutoConfiguration
```

这个类注入了四个组件

ErrorPageCustomizer：发生错误之后，进入错误页面

```java
private static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {

   @Override
   public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
      ErrorPage errorPage = new ErrorPage(this.properties.getServletPrefix()
            + this.properties.getError().getPath());
      errorPageRegistry.addErrorPages(errorPage);
   }
```

this.properties.getError().getPath()值：

```java
@Value("${error.path:/error}")
private String path = "/error";
```

BasicErrorController： 我们可以看到，如果没有配置server.error.path，则默认处理处理的时/error请求，浏览器请求头为text/html，所以返回错误页面，而其他客户端则返回json

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController{
    //产生html类型的数据；浏览器发送的请求来到这个方法处理
     @RequestMapping(produces = "text/html")
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

	@RequestMapping
	@ResponseBody    //产生json数据，其他客户端来到这个方法处理；
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```

所以：

一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被**BasicErrorController**处理；

DefaultErrorViewResolver：

​		返回 默认的 错误View

DefaultErrorAttributes：帮助我们在错误的页面设置一些默认属性

```java
@Override
public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
      boolean includeStackTrace) {
   Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
   errorAttributes.put("timestamp", new Date());
   addStatus(errorAttributes, requestAttributes);
   addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
   addPath(errorAttributes, requestAttributes);
   return errorAttributes;
}
```

### 返回定制的json

第一种方式，这种方式浏览器访问也会返回json

```java
@ControllerAdvice
public class MyExceptionHandler {

    @ResponseBody
    @ExceptionHandler(value = MyException.class)
    public Map handleException(Exception e){
        Map map = new HashMap<>();
        map.put("exception", e.getMessage());
        return map;
    }
}
```

第二种方式

```java
 @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
         * Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        //转发到/error
        return "forward:/error";
    }
```



```java
//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company","atguigu");
        return map;
    }
}
```



## springboot使用 servlet

### 第一种方式

访问：<http://localhost:8080/firstServlet>，能够在控制台看到对应输出

```Java
/**
 * 传统的servlet需要在web.xml中配置servlet-name和path
 * 此处只需要写注解即可
 * Created by Administrator on 2019/5/15.
 */
@WebServlet(name = "FirstServlet", urlPatterns = "/firstServlet")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.print("啦啦啦，");
    }
}
```

```java
@SpringBootApplication
@ServletComponentScan //启动时扫描@WebServlet注解，并将其实例化
public class App {
    public static void main(String [] args){
        SpringApplication.run(App.class,args);
    }
}
```

### 第二种方式

不需要写servlet注解

```java
@Configuration
public class SecondServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.print("第二个啦啦啦，");
    }
}
```

```java
@SpringBootApplication
public class App2 {
    public static void main(String [] args){
        SpringApplication.run(App2.class,args);
    }
}
```

但需要增加一个config配置产生ServletRegistrationBean的bean，将servlet配置其中

```java
@Configuration
public class ServletRegistBeanConfig {

    @Bean //产生一个ServletRegistrationBean的bean
    public ServletRegistrationBean getServletRegisTrationBean(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new SecondServlet());
        servletRegistrationBean.addUrlMappings("/secondServlet");
        return servletRegistrationBean;
    }
}
```
## springboot使用Listner

### 第一种方式

```java
@SpringBootApplication
@ServletComponentScan 
public class App {
    public static void main(String [] args){
        SpringApplication.run(App.class,args);
    }
}
```

```java
@WebListener
public class FirstListner implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("------");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {

    }

}
```

### 第二种方式

```java
@SpringBootApplication
public class App2 {
    public static void main(String [] args){
        SpringApplication.run(App2.class,args);
    }
}
```

```java
public class SecondListner implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("-----");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {

    }
}
```

```java
@Configuration
public class ListnerRegistBeanConfig {

    @Bean
    public ServletListenerRegistrationBean getServletListenerRegistrationBean(){
        ServletListenerRegistrationBean<SecondListner> secondListnerServletListenerRegistrationBean
                = new ServletListenerRegistrationBean<SecondListner>(new SecondListner());
        return secondListnerServletListenerRegistrationBean;
    }
}
```

## filter

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/getyaml2"));
    return registrationBean;
}
```

# spring boot 与 dubbo

## provide端

yml配置

```yaml
dubbo:
  scan: #扫描的service层，如果配置了这个，则buxuy
  		#@EnableDubbo这个注解了
    basePackages: com.xiao.service
  application:
    id: dubbo-xiao-provide
    # 服务名
    name: dubbo-gree-provide
  registry:
    #注册中心
    address: zookeeper://192.168.94.128:2181
  server: true
  protocol:
    name: dubbo #通信协议
    prot: 20880
```

service配置：注解service：import com.alibaba.dubbo.config.annotation.Service;

```java
@Service
public class UserServiceImpl implements UserService {
    @Override
    public TUser getUser() {
        TUser user = new TUser();
        user.setId(1);
        user.setUsername("哈哈哈");
        return user;
    }
}
```

启动类配置，注意通过EnableDubbo注解来开启dubbo注解

```java
@EnableDubbo
@SpringBootApplication
public class Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class, args);
    }
}
```

## consumer端

```java
@RestController
public class HelloWorldController {

    @Reference
    private UserService userService;

    @RequestMapping("/user")
    public TUser getUser(){
        return userService.getUser();
    }
}
```

## 超时配置

消费端：reference默认使用的超时为 consumer的timeout:1000

​				可也以如此配置@Reference(timeout = 6000)

也可在服务端设置timeout

方法级别>类级别

消费端>服务端

## 重试次数

在comsumer端设置重试次数，当超时之后，provide端执行了三次

幂等的可以设置重试次数（如：查询）

```java
@Reference(retries = 2)
```

## 多版本

provide端：

定义两个service提供服务

```java
@com.alibaba.dubbo.config.annotation.Service(version = "2.0.0")
@Service(value ="userService2" )
public class UserService2Impl implements UserService
```

```java
@com.alibaba.dubbo.config.annotation.Service(version = "1.0.0")
@Service
public class UserServiceImpl implements UserService 
```

comsumer端指定版本

```java
@Reference(version = "2.0.0")
private UserService userService;
```

## 本地存根

在consumer端调用provide端的service时，先经过consumer层的service，判断是否符合条件，再调用provide端的service

代码（consumer）:

```java
@Reference(stub = "com.xiao.service.impl.UserServiceImp")
private UserService userService;
```

```java
public class UserServiceImp implements UserService {
    private UserService userService;
    public UserServiceImp(UserService userService) {
        this.userService=userService;
    }

    @Override
    public TUser getUser() throws Exception {
        System.out.println("本地存根");
        return userService.getUser();
    }
}
```

## dubbo直连 

```java
@Reference(url="127.0.0.1:20881")
private UserService userService;
```

## dubbo负载均衡机制

默认是随机负载的

### 服务端服务级别

```xml
<dubbo:service interface="..." loadbalance="roundrobin" />
```

***客户端服务级别***

```xml
<dubbo:reference interface="..." loadbalance="roundrobin" />
```

***服务端方法级别***

```xml
<dubbo:service interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:service>
```

***客户端方法级别***

```xml
<dubbo:reference interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:reference>
```

## 服务降级

服务器压力剧增的情况下，对一些服务进行不处理策略

在dubbo控制台，在消费端

屏蔽：不进行远程调用，直接返回为空

容错：对一些错误不处理（如超时），返回为空

## 容错机制

Failover Cluster

失败自动切换，当出现失败，重试其它服务器 [[1\]](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn1)。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含第一次)。

重试次数配置如下：

```xml
<dubbo:service retries="2" />
```

或

```xml
<dubbo:reference retries="2" />
```

或

```xml
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn2)。通常用于通知所有提供者更新缓存或日志等本地资源信息。

 集群模式配置

按照以下示例在服务提供方和消费方配置集群模式

```xml
<dubbo:service cluster="failsafe" />
```

或

```xml
<dubbo:reference cluster="failsafe" />
```

##  dubbo原理

### 标签解析

spring 接口 BeanDefinitionParser

```java
public class DubboBeanDefinitionParser implements BeanDefinitionParser {
        private static BeanDefinition parse(Element element, ParserContext parserContext, Class<?> beanClass, boolean required) {
        RootBeanDefinition beanDefinition = new RootBeanDefinition();
        beanDefinition.setBeanClass(beanClass);
        beanDefinition.setLazyInit(false);
        String id = element.getAttribute("id");
```



# springboot 与 jdbc

引入pom

```xml
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
</dependency>
```

配置yml

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.94.129:3306/mytest
    driver-class-name: com.mysql.jdbc.Driver
```

测试连接

```java
@Autowired
private DataSource dataSource;
@Test
public void contextLoads() throws Exception {
   System.out.println("连接class:"+dataSource.getClass());
   Connection connection = dataSource.getConnection();
   System.out.println(connection);
   connection.close();
}
```

可以看到，打印出

连接class:class org.apache.tomcat.jdbc.pool.DataSource

可以看到，默认时用tomcatajdbc连接的



原理：如果引入了tomcat的daasource，并且，配置了spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource就使用这个配置（matchIfMissing = true表示没配置也默认为配置了）

```java
abstract class DataSourceConfiguration {
    @ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource", matchIfMissing = true)
	static class Tomcat extends DataSourceConfiguration {
        @Bean
		@ConfigurationProperties(prefix = "spring.datasource.tomcat")
		public org.apache.tomcat.jdbc.pool.DataSource dataSource(
```

如果我们需要使用自定义的数据源，则利用这个配置

```java
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {
   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
       //使用DataSourceBuilder创建数据源，
       //利用反射创建响应type的数据源，并且绑定相关属性
      return properties.initializeDataSourceBuilder().build();
   }
}
```

### jdbc的自动建表

原理：DataSourceInitializer回去初始化建表语句

```java
public class DataSourceAutoConfiguration {
    @Bean
	@ConditionalOnMissingBean
	public DataSourceInitializer dataSourceInitializer(DataSourceProperties properties,
			ApplicationContext applicationContext) {
		return new DataSourceInitializer(properties, applicationContext);
	}

```

**DataSourceInitializer：ApplicationListener**；

	作用：
		1）、runSchemaScripts();运行建表语句；
	
		2）、runDataScripts();运行插入数据的sql语句；

默认只需要将文件命名为：

```properties
schema-*.sql、data-*.sql
默认规则：schema.sql，schema-all.sql；
可以使用   
	schema:
      - classpath:department.sql
      指定位置
```

# 整合Druid数据源

引入jar包

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid</artifactId>
   <version>1.1.10</version>
</dependency>
```

配置配置文件

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.94.129:3306/mytest
    driver-class-name: com.mysql.jdbc.Driver
    #使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    #数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
#   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

这个时候，利用的boot里面的配置反射使用druid的数据源，但是我们发现无法数据源其他配置无法配置到druid的数据源中，所以，我们不能使用默认的反射使用druid的数据源，需要自定义config来进行配置

```java
@Configuration
public class DruidConfig {
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource getDataSource(){
        //druiddatasource里面initialSize的配置属性，所以
        return new DruidDataSource();
    }
    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean 
            = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();
        // 用户名
        initParams.put("loginUsername","admin");
        // 密码
        initParams.put("loginPassword","123456");
        //IP白名单 默认就是允许所有访问
        initParams.put("allow","192.168.1.72,127.0.0.1");
        // IP黑名单 (存在共同时，deny优先于allow)
        initParams.put("deny","");
        bean.setInitParameters(initParams);
        return bean;
    }
    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        WebStatFilter webStatFilter = new WebStatFilter();
        bean.setFilter(webStatFilter);
        Map<String,String> initParams = new HashMap<>();
        //忽略资源
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return  bean;
    }
}
```

访问：<http://127.0.0.1:8080/druid/login.html>

# 与mybatis整合

## 单数据源方式

配置yml

```yml
spring:
  datasource:
    url: jdbc:mysql://192.168.94.139:3306/mytest
    driverClassName: com.mysql.jdbc.Driver
    username: test
    password: test=123456
    #配置数据源
    type: com.alibaba.druid.pool.DruidDataSource

mybatis:
  # 给实体类的包目录起别名
  type-aliases-package: com.xiao.entry
  mapper-locations: classpath:mapper/*.xml
```

在：resources/mapper/UserMapper.xml下建立mapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.xiao.mapper.UserMapper" >
    <resultMap id="BaseResultMap" type="com.xiao.entry.TUser" >
        <id column="id" property="id" jdbcType="VARCHAR" />
        <result column="name" property="name" jdbcType="VARCHAR" />
    </resultMap>

    <sql id="Base_Column_List" >
        id, name
    </sql>

    <select id="getAll" resultMap="BaseResultMap">
        select
          <include refid="Base_Column_List"></include>
        from users
    </select>

</mapper>
```

application.java类

```java
@SpringBootApplication
@MapperScan("com.xiao.mapper") //扫描mybatis的mapper接口
public class Application {
    public static void main(String [] args){
        SpringApplication.run(Application.class,args);
    }
}
```

## 注解版

引入jar包

```xml
<dependency>
   <groupId>org.mybatis.spring.boot</groupId>
   <artifactId>mybatis-spring-boot-starter</artifactId>
   <version>1.3.1</version>
</dependency>
```

mapper:如果在**启动类**加入@MapperScan(value = "com.xiao.Mapper")，则不需要@Mapper

```java
@Mapper 
public interface UserMapper {
    @Select("Select * from t_user where username=#{username}")
    public List<UserBean> findUsers(String username) throws Exception;

    //以id自增
    @Options(useGeneratedKeys=true, keyProperty = "id")
    @Insert("insert into t_user(username) values(#{username})")
    public void insertUser(UserBean userBean) throws Exception;

    @Delete("delete t_user where username=#{username}")
    public void deleteUser(String username);

    @Update("update t_user set username=#{username} where id=#{id}")
    public void updateUser(UserBean userBean) throws Exception;
}
```

## xml配置版

mapper

```java
@Mapper
public interface UserNameMapper {
    public UserBean getUser(String id) throws Exception;
}
```

sql映射文件：mybatis/Mapper/UserMapper1.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xiao.Mapper.UserNameMapper">

    <select id="getUser" resultType="com.xiao.bean.UserBean">
        SELECT * FROM t_user WHERE id=#{id}
    </select>
</mapper>
```

mybatis配置文件：mybatis/mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!--驼峰命名法配置-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

配置文件：

```yaml
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件位置
  mapper-locations: classpath:mybatis/Mapper/*.xml
```

# 启动配置原理

配置在META-INF/spring.factories

**ApplicationContextInitializer**

**SpringApplicationRunListener**



只需要放在ioc容器中

**ApplicationRunner**

**CommandLineRunner**

## 启动流程

1 启动进入run方法，sources为启动的application那个main方法类

```java
ublic static ConfigurableApplicationContext run(Object[] sources, String[] args) {
   return new SpringApplication(sources).run(args);
}
```

2 之后main方法进入new SpringApplication的构造方法中

```java
public SpringApplication(Object... sources) {
   initialize(sources);
}
```

3 

```java
private void initialize(Object[] sources) {
    //保存main方法启动类
   if (sources != null && sources.length > 0) {
      this.sources.addAll(Arrays.asList(sources));
   }
    //判断javax.servlet.Servlet是否存在
    //来判断是否时web应用
   this.webEnvironment = deduceWebEnvironment();
   //通过SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    //来获取META-INF/spring.factories ApplicationContextInitializer配置类
   setInitializers((Collection) getSpringFactoriesInstances(
         ApplicationContextInitializer.class));
    //来获取META-INF/spring.factories ApplicationListener配置类
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //从多个配置类中找到有main方法的主配置类
   this.mainApplicationClass = deduceMainApplicationClass();
}
```

4 运行run方法

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
     //获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
       //封装命令行参数(main方法传来的args)
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
       //创建环境完成后回调SpringApplicationRunListener.environmentPrepared()；
       //表示环境准备完成
      Banner printedBanner = printBanner(environment);
       //创建ApplicationContext；决定创建web的ioc还是普通的ioc
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
       //准备上下文环境;将environment保存到ioc中；
       //applyInitializers()：回调之前保存的所有的
       //ApplicationContextInitializer的initialize方法
       //回调所有的SpringApplicationRunListener的contextPrepared()；
       //
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       //prepareContext运行完成以后回调
       //所有的SpringApplicationRunListener的contextLoaded（）；
       //s刷新容器；ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）；Spring注解版
       //扫描，创建，加载所有组件的地方；（配置类，组件，自动配置）
      refreshContext(context);
       //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
       //ApplicationRunner先回调，CommandLineRunner再回调
      afterRefresh(context, applicationArguments);
       //所有的SpringApplicationRunListener回调finished方法
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
       //整个SpringBoot应用启动完成以后返回启动的ioc容器
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

# 自定义start

启动器只用来做依赖导入；

专门来写一个自动配置模块；

启动器依赖自动配置；别人只需要引入启动器（starter）

mybatis-spring-boot-starter；自定义启动器名-spring-boot-starter

## 步骤

- 建立一个空的maven项目

  这个项目是用来引导Autoconfigration的项目，一般命名为 自定义启动器名-spring-boot-starter 
  
  ```xml
  <groupId>com.xiao</groupId>
      <artifactId>xiao-spring-boot-starter</artifactId>
      <packaging>jar</packaging>
      <version>1.0-SNAPSHOT</version>
      <modelVersion>4.0.0</modelVersion>
      <dependencies>
          <dependency>
              <groupId>com.xiao</groupId>
              <artifactId>xiao-spring-boot-starter-autoconfigurer</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
      </dependencies>
  ```
  
- 建立自动配置的maven 类
```xml
<groupId>com.xiao</groupId>
    <artifactId>xiao-spring-boot-starter-autoconfigurer</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/>
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!--引入spring-boot-starter；所有starter的基本配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
```

- 建立config类，读取配置文件的数据

```java
//定义一个config，读取配置文件的数据
@ConfigurationProperties(prefix = "xiao.hello")
public class HelloConfig {
    private String name;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

- 建立service类，用来操作
```java
public class HelloService {
    private HelloConfig helloConfig;
    public HelloService(HelloConfig helloConfig){
        this.helloConfig=helloConfig;
    }

    public String getName(){
        return "经过service的类："+helloConfig.getName();
    }
}
```
- 建立自动配置的类
```java
@Configuration
@ConditionalOnWebApplication//在web应用中自动配置
@EnableConfigurationProperties(HelloConfig.class)//使Config生效,并且注入到容器中
public class HelloStartAutoConfiguration {

    @Autowired
    private HelloConfig helloConfig;

    @Bean
    public HelloService getHelloService(){
        HelloService helloService = new HelloService(helloConfig);
        return helloService;
    }
}
```
- 建立META-INF/spring.factories文件
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    com.xiao.starter.HelloStartAutoConfiguration
```

# spring boot 与 缓存

## JSR107

Java Caching定义了5个核心接口，分别是**CachingProvider**, **CacheManager**, **Cache**, **Entry** 和 **Expiry**。


•**CachingProvider**定义了创建、配置、获取、管理和控制多个**CacheManager**。一个应用可以在运行期访问多个CachingProvider。


•**CacheManager**定义了创建、配置、获取、管理和控制多个唯一命名的**Cache**，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。如：管理redis缓存等。。。。


•**Cache**是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。如A cache缓存员工， B cache 缓存部门


•**Entry**是一个存储在Cache中的key-value对。


•**Expiry** 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。


## spring 缓存管理

因为jsr107实现比较复杂，所以，spring只保留了cachemanager和cache来管理缓存.

cachemanager来管理cache，cache来操作真正的缓存

## 缓存注解

| **Cache**          | 缓存接口，定义缓存操作。实现有：**RedisCache**、**EhCacheCache**、**ConcurrentMapCache**等 |
| ------------------ | :----------------------------------------------------------- |
| **CacheManager**   | **缓存管理器，管理各种缓存（****Cache****）组件**            |
| **@Cacheable**     | **主要针对方法配置，能够根据方法的请求参数对其结果进行缓存**  （查询值的时候，对返回结果进行缓存） |
| **@CacheEvict**    | **清空缓存**                                                 |
| **@CachePut**      | **保证方法被调用，又希望结果被缓存。**   （新增值的时候，对新增的内容进行缓存） |
| **@EnableCaching** | **开启基于注解的缓存**                                       |
| **keyGenerator**   | **缓存数据时****key生成策略**                                |
| **serialize**      | **缓存数据时****value序列化策略**                            |

## 使用缓存

### 原理

CacheAutoConfiguration类进行自动配置

```java
@Configuration
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureBefore(HibernateJpaAutoConfiguration.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
      RedisAutoConfiguration.class })
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration 
```

CacheConfigurationImportSelector会导入一系列缓存的配置

```java
static class CacheConfigurationImportSelector implements ImportSelector {
   @Override
   public String[] selectImports(AnnotationMetadata importingClassMetadata) {
      CacheType[] types = CacheType.values();
      String[] imports = new String[types.length];
      for (int i = 0; i < types.length; i++) {
         imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
      }
      return imports;
   }
}
```

imports中的值为这些

```java
mappings.put(CacheType.GENERIC, GenericCacheConfiguration.class);
mappings.put(CacheType.EHCACHE, EhCacheCacheConfiguration.class);
mappings.put(CacheType.HAZELCAST, HazelcastCacheConfiguration.class);
mappings.put(CacheType.INFINISPAN, InfinispanCacheConfiguration.class);
mappings.put(CacheType.JCACHE, JCacheCacheConfiguration.class);
mappings.put(CacheType.COUCHBASE, CouchbaseCacheConfiguration.class);
mappings.put(CacheType.REDIS, RedisCacheConfiguration.class);
mappings.put(CacheType.CAFFEINE, CaffeineCacheConfiguration.class);
addGuavaMapping(mappings);
mappings.put(CacheType.SIMPLE, SimpleCacheConfiguration.class);
mappings.put(CacheType.NONE, NoOpCacheConfiguration.class);
```

默认：SimpleCacheConfiguration这个类生效,这个类给容器注入一个ConcurrentMapCacheManager

可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；

```java
@Configuration
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class SimpleCacheConfiguration {
    @Bean
	public ConcurrentMapCacheManager cacheManager() {
		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}
```

key使用keyGenerator生成的，默认是SimpleKeyGenerator

​	SimpleKeyGenerator:

​		如果没有参数：key== new SimpleKey()

​		如果一个参数：key=
Object param = params[0];
if(param != null && !param.getClass().isArray()) {
​    return param;
}

​		如果多个参数：key=new SimpleKey(params);

**key的写法：**

-  #i d，获取参数id的值，#a0  #p0  #root.args[0] 第几个参数
- SpEL表达式：
- 自己指定keyGenerator，生成key

 注入容器myKeyGeneratorbean，在方法上注解keyGenerator = "myKeyGenerator"

```
@Bean("myKeyGenerator")
public KeyGenerator keyGenerator(){
    return new KeyGenerator(){
        @Override
        public Object generate(Object target, Method method, Object... params) {
            return method.getName()+ Arrays.asList(params).toString();
        }
    };
}
```

```java
@Cacheable(value = {"users"}, keyGenerator = "myKeyGenerator")
public TUser getUser(Integer id){
    System.out.println("未使用缓存:"+id);
    return userMapper.getUserById(id);
}
```

### 步骤

导入jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

使用注解开启缓存

```java
@SpringBootApplication
@MapperScan("com.xiao.mapper") //扫描mybatis的mapper接口
@EnableCaching//开启缓存
public class Application {
    public static void main(String [] args){
        SpringApplication.run(Application.class,args);
    }
}
```

缓存在service层使用

 使用 在方法上

属性：

- cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；(其实就是给cachemanager下的cache命名)
- key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  arg0-方法的返回值
  
- keyGenerator：key的生成器；可以自己指定key的生成器的组件id
             key/keyGenerator：二选一使用;
- cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
- condition：指定符合条件的情况下才缓存；
          condition = "#id>0" 
          condition = "#a0>1"：第一个参数的值>1的时候才进行缓存
- unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行                判断
       unless = "#result == null"
      unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
-  sync：是否使用异步模式

```java
@Transactional(readOnly = true)
@Cacheable(value = {"users"})
public TUser getUser(Integer id){
    System.out.println("未使用缓存:"+id);
    return userMapper.getUserById(id);
}
```

## CachePut

更新缓存，一般用于修改或新增后，去更新刷新对应value的key值的缓存

 key = "#tUser.id":使用传入的参数的员工id；
          key = "#result.id"：使用返回后的id
          **@Cacheable的key是不能用#result**

```java
@CachePut(value = {"users"}, key = "#tUser.id")
public TUser updateUser(TUser tUser){
    return tUser;
}
```

## CacheEvict

删除缓存

key：指定要清除的数据
- allEntries = true：指定清除这个缓存中所有的数据
- beforeInvocation = false：缓存的清除是否在方法之前执行
	默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除

-  beforeInvocation = true：
   	代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除

```java
@CacheEvict(value = {"users"}, key = "#id")
public void deleteUser(Integer id){

}
```

## CacheConfig

指定该类使用的全局配置

```java
@Service
@CacheConfig(cacheNames= {"users"})
public class UserServiceImpl {
```

## 使用redis作为缓存

### 原理

只要引入redis的jar包，RedisAutoConfiguration就会生效,缓存也会使用redis作为缓存

```java
@Configuration
@ConditionalOnClass({ JedisConnection.class, RedisOperations.class, Jedis.class })
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfiguration {
```

它加入了RedisTemplate注入了容器，这两个是用来操作redis的

```java
@Configuration
protected static class RedisConfiguration {

   @Bean
   @ConditionalOnMissingBean(name = "redisTemplate")
   public RedisTemplate<Object, Object> redisTemplate(
         RedisConnectionFactory redisConnectionFactory)
               throws UnknownHostException {
      RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
      template.setConnectionFactory(redisConnectionFactory);
      return template;
   }

   @Bean
   @ConditionalOnMissingBean(StringRedisTemplate.class)
   public StringRedisTemplate stringRedisTemplate(
         RedisConnectionFactory redisConnectionFactory)
               throws UnknownHostException {
      StringRedisTemplate template = new StringRedisTemplate();
      template.setConnectionFactory(redisConnectionFactory);
      return template;
   }

}
```

## 使用redis 的客户端

引入redis-starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置地址

```yaml
spring:
  #redis 连接
  redis:
    host: 192.168.94.129
```

注入操作string类型的客户端

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;
@org.junit.Test
public void test(){
    stringRedisTemplate.opsForValue().set("111", "222");
}
```

## 自定义序列化工具

默认使用的时：JdkSerializationRedisSerializer序列化工具，他们都实现了RedisSerializer接口

建立配置类，配置json序列化自定义

```java
@Configuration
public class MyRedisConfig {
    @Bean(value = "useRedisTemplate")
    public RedisTemplate<Object, TUser> useRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, TUser> template = new RedisTemplate<Object, TUser>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer
                = new Jackson2JsonRedisSerializer(TUser.class);
        template.setDefaultSerializer(jackson2JsonRedisSerializer);
        return template;
    }
}
```

使用自定义配置

```java
@Autowired
private RedisTemplate<Object, TUser> useRedisTemplate;
@org.junit.Test
public void test2(){
   TUser tUser = new TUser();
   tUser.setName("张三");
   tUser.setId(1);
   useRedisTemplate.opsForValue().set("user", tUser);
}
```

查看redis对应键值

```json
{
  "id": 1,
  "name": "张三"
}
```

## 2.x 与1.5的区别

### 默认客户端

| spring-boot版本 | 默认客户端类型 |
| :-------------- | :------------- |
| 1.5.x           | jedis          |
| 2.x             | lettuce        |

在1.5.x中，我们配置jedis连接池只需要配置 **spring.redis.pool.*** 开始的配置即可，如下配置

```
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.pool.min-idle=0
spring.redis.pool.max-idle=8
```

但在2.x版本中由于引入了不同的客户端，需要指定配置哪种连接池，如下配置

```
#jedis客户端
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.min-idle=0
spring.redis.jedis.pool.max-idle=8
#lettuce客户端
spring.redis.lettuce.pool.min-idle=0
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.shutdown-timeout=100ms
```



### JavaConfig方式配置

1.5

#### CacheManager配置

```java
@Bean
public CacheManager cacheManager(RedisTemplate redisTemplate) {
    RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
    //默认超时时间，单位秒
    redisCacheManager.setDefaultExpiration(60);
    //缓存超时时间Map，key为cacheName，value为超时,单位是秒
    Map<String, Long> expiresMap = new HashMap<>();
    //缓存用户信息的cacheName和超时时间
    expiresMap.put("user", 1800L);
    //缓存产品信息的cacheName和超时时间
    expiresMap.put("product", 600L);
    redisCacheManager.setExpires(expiresMap);
    return redisCacheManager;
}
```

#### cache调用代码

```java
@Cacheable(value = "user", key = "'user:'+#id", unless = "#result==null")
public String getUser(int id) {
    //逻辑操作
}
@Cacheable(value = "product", key = "'product:'+#id", unless = "#result==null")
public String getProduct(int id) {
    //逻辑操作
}
```



#### CacheManager配置

2.x版本开始，代码方式配置变化比较大，同时增加了更多配置参数

```java
@Bean
CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
    //user信息缓存配置
    RedisCacheConfiguration userCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(30)).disableCachingNullValues().prefixKeysWith("user");
    //product信息缓存配置
    RedisCacheConfiguration productCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(10)).disableCachingNullValues().prefixKeysWith("product");
    Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();
    redisCacheConfigurationMap.put("user", userCacheConfiguration);
    redisCacheConfigurationMap.put("product", productCacheConfiguration);
    //初始化一个RedisCacheWriter
    RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(connectionFactory);
    
    
    //设置CacheManager的值序列化方式为JdkSerializationRedisSerializer,但其实RedisCacheConfiguration默认就是使用StringRedisSerializer序列化key，JdkSerializationRedisSerializer序列化value,所以以下注释代码为默认实现
    //ClassLoader loader = this.getClass().getClassLoader();
    //JdkSerializationRedisSerializer jdkSerializer = new JdkSerializationRedisSerializer(loader);
    //RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair.fromSerializer(jdkSerializer);
    //RedisCacheConfiguration defaultCacheConfig=RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(pair);
    
    
    RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig();
    //设置默认超过期时间是30秒
    defaultCacheConfig.entryTtl(Duration.ofSeconds(30));
    //初始化RedisCacheManager
    RedisCacheManager cacheManager = new RedisCacheManager(redisCacheWriter, defaultCacheConfig, redisCacheConfigurationMap);
    return cacheManager;
}
```

以上代码中RedisCacheConfiguration类为2.x新增的配置类，增加了几个配置项。这里比较奇怪的是调用它的配置方法每一次都会重新生成一个配置对象，而不是在原对象上直接修改参数值，这一点本人没搞懂作者为何要选择这种方式。

#### cache调用代码

```java
@Cacheable(value = "user", key = "#id", unless = "#result==null")
public String getUser(int id) {
    //逻辑操作
}
@Cacheable(value = "product", key = "#id", unless = "#result==null")
public String getProduct(int id) {
    //逻辑操作
}
```

# spring boot 与消息

消息队列主要有两种形式的目的地

1.队列（queue）：点对点消息通信（point-to-point）


2.主题（topic）：发布（publish）/订阅（subscribe）消息通信

## 概念

点对点式：
– 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列 
– 消息只能被一个客户端接收，但可以有多个客户端


发布订阅式：
– 发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

JMS（Java Message Service）JAVA消息服务：
– 基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

AMQP（Advanced Message Queuing Protocol）
– 高级消息队列协议，也是一个消息代理的规范，兼容JMS
– RabbitMQ是AMQP的实现

# spring boot与索引



# 定时任务

Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、 TaskScheduler 接口 

```java
@EnableAsync  //开启异步注解功能,开启这个，可以借助异步处理，来实现多线程并发跑任务
@EnableScheduling //开启基于注解的定时任务
@SpringBootApplication
public class Application {
```

cron表达式

second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）

 0 * * * * MON-FRI

【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次

【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次

【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次

【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次

【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；

```java
@Service
public class ScheduledService {
  	@Asyn //没有这个注解，多个定时器会在同一以同一个线程来跑
    @Scheduled(cron = "0/4 * * * * MON-SAT")  //每4秒执行一次
    public void hello(){
```


# Spring概述

## spring特性

### 核心特性（core）

- Ioc容器
- Spring 事件
- 资源管理
- 国际化
- 校验
- 数据绑定
- 类型转换
- spring 表达式
- 面向切面编程
  - aop

### web技术

- web servlet
- web reactive

### 测试

- 模拟对象
- TestContext框架
  - 加载spring的上下文

## spring 版本特性

- 1.x
- 2.x
- 3.x
  - 引入注解
  - 基本确定内核
- 4.x
  - 对spring boot 1.x的支持
- 5.x

## spring 模块设计

- spring-aop

- spring-core
  - 

- spring-jcl
  - 日志的支持

...

## spring 对jdk api支持

- java5

| XML处理(DOM,SAX...) | 1.0  | XmlBeanDefinitionReader |
| ------------------- | ---- | ----------------------- |
| Java管理扩展(JMX)   | 1.2  | @ManagedResource        |
| 并发框架(J.U.C)     | 3.0  | ThreadPoolTaskExecutor  |
| 格式化(Formatter)   | 3.0  | DateFormat              |

- java6

| JDBC4.0                      | 1.0  | JdbcTemplate                         |
| ---------------------------- | ---- | ------------------------------------ |
| Common Annotations(JSR 250)  | 2.5  |                                      |
| 可插拔注解处理api（JSR 269） | 5.0  | @Indexed(减少运行时的scanning的操作) |
|                              |      |                                      |

## Spring编程模型

- aware接口
  - aware回调接口，每当初始化这个接口的实现bean时，会回调设置一个值
  - 如：ApplicationContextAware.setApplicationContext
  - 如：
- 组合模式
  - Composite
- 模板模式

# spring核心模块

spring-core: spring基础api，如资源管理，泛型处理

spring-beans：依赖注入，依赖查找

spring-aop: 动态代理，字节码提升

spring-context:事件驱动，注解驱动，模块驱动

spring-expression:spring表达式语言



# Spring IOC

## IOC容器的职责

- 实现与应用解耦

- 依赖处理
  - 依赖查找：如通过名称去查找
  - 依赖注入

## Java Beans

```tex
JavaBean是一种特殊的类，主要用于传递数据信息，
这种类中的方法主要用于访问私有的字段，
且方法名符合某种命名规则。
如果在两个模块之间传递信息，
可以将信息封装进JavaBean中，
这种对象称为“值对象”(Value Object)，或“VO”。
方法比较少。
这些信息储存在类的私有变量中，通过set()、get()获得。
```

### 特性

- 依赖查找
- 生命周期管理
- 配置元信息
- 事件
- 资源管理
- 持久化

### JDK内省类库

- 定义一个pojo类

```java
@Getter
@Setter
@ToString
public class Person {
    private String name;
    private Integer age;
}
```

> Introspector类

- 编写BeanInfo的示例
  - 可以看到打印的，多出了一个
  - name=class; propertyType=class java.lang.Class readMethod=public
  - 那是因为顶层Object类有一个getClass方法，他会默认以为这个是一个**可读的方法**

```java
public static void main(String[] args) throws IntrospectionException {
        BeanInfo beanInfo = Introspector.getBeanInfo(Person.class);
    	//getPropertyDescriptors()，获得属性的描述，
    	//可以采用遍历BeanInfo的方法，来查找、设置类的属性
        Arrays.stream(beanInfo.getPropertyDescriptors()).forEach(propertyDescriptor -> {
           System.out.println(propertyDescriptor.toString());
        });
    }
```

- 打印出Person的属性信息,和**父类object信息**

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/java/spring/20210609175925.png)

- 我们可以加上stopClass的参数，避免他向上寻找父类

```java
BeanInfo beanInfo = Introspector.getBeanInfo(Person.class, Object.class);
```

## 依赖查找

### 延迟查找与及时查找

- 定义配置类

```java
@Configurable
public class DependencyLookUpConfig {
    @Bean
    public Person person() {
        return new Person();
    }

    @Bean
    public ObjectFactoryCreatingFactoryBean objectFactory() {
        ObjectFactoryCreatingFactoryBean objectFactory = new ObjectFactoryCreatingFactoryBean();
        //将bean的名称设置进去
        objectFactory.setTargetBeanName("person");
        return objectFactory;
    }
}
```

> 依赖查找

- 及时查找

```java
public static void main(String[] args) {
    BeanFactory beanFactory = new AnnotationConfigApplicationContext(DependencyLookUpConfig.class);
    lookUpLazyTime(beanFactory);
    lookUpRealTime(beanFactory);
}

/**
     * 及时查找
     * @param beanFactory
     */
static void lookUpRealTime(BeanFactory beanFactory) {
    Person bean = beanFactory.getBean(Person.class);
    System.out.println(bean);
}
```

- 延迟查找:并不是里面初始化,延迟查找需要通过中间类来获取bean

```java
/**
     * 延迟查找
     * @param beanFactory
     */
static void lookUpLazyTime(BeanFactory beanFactory) {
    ObjectFactory<Person> bean = beanFactory.getBean(ObjectFactory.class);
    System.out.println(bean.getObject());
}
```

### 通过注解来查找bean

- 定义一个AnUser额注解
- 将注解标注在SuperPerson类上

```java
@AnUser
public class SuperPerson extends Person {
}
```

- 通过注解获取对应的bean

```java
public static void main(String[] args) {
    BeanFactory beanFactory = new AnnotationConfigApplicationContext(DependencyLookUpConfig.class);
    lookUpByAnnotation(beanFactory);
}

/**
 * 通过注解来查找bean
 * @param beanFactory
 */
private static void lookUpByAnnotation(BeanFactory beanFactory) {
    if(beanFactory instanceof ListableBeanFactory) {
        Map<String, Object> beans = ((ListableBeanFactory) beanFactory).getBeansWithAnnotation(AnUser.class);
        System.out.println(beans.toString());
    }
}
```

## 依赖注入

- 依赖注入来源与依赖查找的来源并不是同一个

## IOC依赖来源

- 自定义bean：我们自定义的bean
- 内建的bean
- 容器内建依赖：beanfactory
-  IoC中，依赖查找和依赖注入的数据来源并不一样。因为BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext这四个并不是Bean，它们只是一种特殊的依赖项，无法通过依赖查找的方式来获取，只能通过依赖注入的方式来获取。

> ApplicationContext

- applicationContext是BeanFactory子接口
- 他提供了获取上下文，监听的方法

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/java/spring/20210128221447.png)

- 查看源码可以得知，application有个getBeanFactory的方法
- 他将BeanFactory组合进来了，所以，applicationContext虽然实现了BeanFactory,但他们是两个东西，一般我们需要beanfactory时，通常用ApplicationContext.getBeanFactory()

> BeanFactory与FactoryBean

-  BeanFactory是IOC最基本的容器，负责管理bean，它为其他具体的IOC容器提供了最基本的规范
- FactoryBean是创建bean的一种方式，帮助实现负责的初始化操作

## IOC生命周期

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {

        prepareRefresh();
        //创建beanfactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        //对beanfactory进行初步的初始化操作
        //加入一些bean依赖，和内建的非bean的依赖
        prepareBeanFactory(beanFactory);

        try {
            postProcessBeanFactory(beanFactory);

            invokeBeanFactoryPostProcessors(beanFactory);

            //对bean的拓展和修改
            registerBeanPostProcessors(beanFactory);
            //国际化操作
            initMessageSource();

            initApplicationEventMulticaster();

            onRefresh();

            registerListeners();


            finishBeanFactoryInitialization(beanFactory);
            finishRefresh();
        }
```

# Spring Bean

## BeanDefinition

- 一个定义bean的元信息的接口
- 用于保存 Bean 的相关信息，包括属性、构造方法参数、依赖的 Bean 名称及是否单例、延迟加载等
- 它是实例化 Bean 的原材料，Spring 就是根据 BeanDefinition 中的信息实例化 Bean
- 这个接口有setter，getter方式来进行操作

> 元信息的一些属性

| 属性(Property)           | 说明                                        |
| ------------------------ | ------------------------------------------- |
| Name                     | Bean的名称或者ID                            |
| Class                    | Bean全类名,必须是具体类，不能用抽象类或接口 |
| Scope                    | Bean的作用域(如: singleton、prototype 等)   |
| Constructor arguments    | Bean构造器参数（用于依赖注入)               |
| Properties               | Bean属性设置（用于依赖注入)                 |
| Autowiring mode          | Bean自动绑定模式(如:通过名称byName)         |
| Lazy initialization mode | Bean延迟初始化模式(延迟和非延迟)            |
| Initialization method    | Bean初始化回调方法名称                      |
| Destruction method       | Bean销毁回调方法名称                        |

> > 定义元信息

```java
BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(Person.class);
//属性设置 第一种方式
beanDefinitionBuilder.addPropertyValue("name", "张三").addPropertyValue("age", 12);
//获取实例  beanDefinition不是bean的最终形态，不是生命周期，可以随时修改
AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();

//通过abstractBeanDefinition 获取beanDefinition
GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
genericBeanDefinition.setBeanClass(Person.class);
MutablePropertyValues propertyValues = new MutablePropertyValues();
propertyValues.add("name", "张三").add("age", 12);
genericBeanDefinition.setPropertyValues(propertyValues);
```

## 命名SpringBean

- Bean的名称
  - bean名称在所在的beanFactory或者他的beanDefinition里是唯一的，而不是在应用里唯一

## 注入容器的方式

> xml方式

<bean name></bean>

> 注解方式

- @Bean
- @Component

> Java API方式

1. 定义相关类的元信息

2. 将元信息注入进入容器中

- 命名的方式： registry.registerBeanDefinition(name, beanDefinitionBuilder.getBeanDefinition());
- 非命名方式： BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinitionBuilder.getBeanDefinition(), registry);

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AnnotationBeanDefinitionDemo.class);

    //命名的方式注册
    registryBeanDefinition(applicationContext, "my-person");
    //非命名的方式
    registryBeanDefinition(applicationContext);

    //获取bean信息
    System.out.println(applicationContext.getBeansOfType(Person.class));
    applicationContext.close();
}

private static void registryBeanDefinition(BeanDefinitionRegistry registry, String name) {
    //1. 定义相关类的元信息
    BeanDefinitionBuilder beanDefinitionBuilder 
        = BeanDefinitionBuilder.genericBeanDefinition(Person.class);
    beanDefinitionBuilder
        .addPropertyValue("name", "张三")
        .addPropertyValue("age", 12);
    if(StringUtils.isEmpty(name)) {
        //2.将元信息注入进入容器中
        BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinitionBuilder.getBeanDefinition(), registry);
        return;
    }
    //2.将元信息注入进入容器中
    registry.registerBeanDefinition(name, beanDefinitionBuilder.getBeanDefinition());

}

private static void registryBeanDefinition(BeanDefinitionRegistry registry) {
    registryBeanDefinition(registry, null);
}
```

- 日志显示
  - 可以看到，非命名的方式#0，带了序号

```log
{my-person=Person(name=张三, age=12), com.xiao.pojo.Person#0=Person(name=张三, age=12)}
```

## BeanDefinition 合并

- 当子类注入bean，父类也注入了bean，那么采用合并的方式，能够将父类的值合并到子类

- RootBeanDefinition表示顶层bean，这是不需要合并的

- 子类的有GenericBeanDefinition，这个是需要合并的

  

> 在ConfigurableBeanFactory#getMergedBeanDefinition会递归的向上合并

```java
//当前不包含这个beandefiniton，则去父类查找是否存在bean
if (!containsBeanDefinition(beanName) 
    && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
   return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(beanName);
}
//如果存在这个bean的话，则继续寻找
return getMergedLocalBeanDefinition(beanName);
```

- 合并后会由GenericBeanDefinition --->RootBeanDefinition

## Bean 初始化

- 注解方式

```java
@PostConstruct
public void postInit() {
    System.out.println("==> PostConstruct init");
}
```

- bean方式

```java
@Bean(initMethod = "initMethod")
public Person person() {
    return new Person();
}
```

```java
public class Person{
    public void initMethod() {
        System.out.println("==> init method");
    }
}
```

- 实现InitializingBean方式

```java
public class Person implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("==> afterPropertiesSet init");
    }
}
```

> 打印顺序

PostConstruct---->afterPropertiesSet----->initMethod

## 层次性依赖查找

- HierarchicalBeanFactory可以类似双亲委派一样，当local beanFactory没有找到bean，则去parent寻找
- 层次性依赖查找接口：HierarchicalBeanFactory
> 根据bean名称查找

1. 基于containsLocalBean方式实现
2. spring api  没有实现，需要我们自己跟进localBean来实现

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext
            = new AnnotationConfigApplicationContext(DependencyLookUpConfig.class);

    // 获取HierarchicalBeanFactory
    // HierarchicalBeanFactory <-- ConfigurableBeanFactory <-- ConfigurableListableBeanFactory
    //所以此处我们只需要获取ConfigurableListableBeanFactory即可
    ConfigurableListableBeanFactory beanFactory = applicationContext.getBeanFactory();
    System.out.println("parent bean factory: "+ beanFactory.getParentBeanFactory());
    //configurable代表可修改的，这里我们去修改parent BeanFactory
    beanFactory.setParentBeanFactory(getBeanFactory());
    System.out.println("parent bean factory: "+ beanFactory.getParentBeanFactory());
}

public static BeanFactory getBeanFactory() {
    AnnotationConfigApplicationContext applicationContext
            = new AnnotationConfigApplicationContext(DependencyLookUpConfig.class);
    ConfigurableListableBeanFactory beanFactory = applicationContext.getBeanFactory();
    return beanFactory;
}
```

## 延迟依赖查找

> ObjectFactory

```java
public interface ObjectFactory<T> {
	T getObject() throws BeansException;
}
```

只是一个普通的对象工厂接口。在Spring中主要两处用了它

1. Scope接口中的get方法，需要传入一个`ObjectFactory`

> ObjectProvider

- ObjectProvider  继承 ObjectFactory

- ObjectProvider在java8的拓展

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(ObjectProviderDemo.class);
    lookupIfAvailable(applicationContext);
    applicationContext.close();
}

/**
 * 如果bean不存在则创建（注意不会注入容器中）
 * @param applicationContext
 */
private static void lookupIfAvailable(AnnotationConfigApplicationContext applicationContext) {
    ObjectProvider<Person> beanProvider = applicationContext.getBeanProvider(Person.class);
    Person person = beanProvider.getIfAvailable(Person::new);
    System.out.println(person);
}
```

- ObjectProvider的集合操作
  - 定义两个String类型的bean
  - 用过provider循环输出

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(ObjectProviderDemo.class);
    lookupIterable(applicationContext);
    applicationContext.close();
}

private static void lookupIterable(AnnotationConfigApplicationContext applicationContext) {
    ObjectProvider<String> beanProvider = applicationContext.getBeanProvider(String.class);
    for(String bean : beanProvider) {
        System.out.println(bean);
    }
}
@Bean
public String getMessage() {
    return "message";
}

@Bean
public String getHello() {
    return "hello";
}
```

## 内建可查找的依赖

- AbstractApplicationContext 内建可查找的依赖

| Bean名称                    | Bean 实例                       | 使用场景             |
| --------------------------- | ------------------------------- | -------------------- |
| environment                 | Enviroment对象                  | 外部配置以及profiles |
| SystemProperties            | Properties对象                  | java系统属性         |
| systemEnvironment           | Map对象                         | 操作系统环境变量     |
| messageSource               | MessageSource对象               | 国际化文案           |
| applicationEventMulticaster | applicationEventMulticaster对象 | Spring 事件广播      |

- AutowiredAnnotationBeanPostProcessor
  - 通过源码可以看到：他处理Autowired，Value相关的注解

```java
public AutowiredAnnotationBeanPostProcessor() {
    this.autowiredAnnotationTypes.add(Autowired.class);
    this.autowiredAnnotationTypes.add(Value.class);
```

- AnnotationConfigUtils

  - 在给定的注册表中注册所有相关的注释后置处理器

## 常见异常

| 异常类型                      | 触发条件(举例)              | 场景举例                                      |
| ----------------------------- | --------------------------- | --------------------------------------------- |
| NoSuchBeanDefinitionException | 当查找Bean不存在于loC容器时 | BeanFactory#getBean、ObjectFactoryt#getObject |
|                               |                             |                                               |
|                               |                             |                                               |

## ObjectFactory BeanFactory区别

答:ObjectFactory 与 BeanFactory 均提供依赖查找的能力。

不过ObjectFactory仅关注一个或一种类型的 Bean依赖查找，开且自身不具备依赖查找的能力，能力则由BeanFactory 输出。

BeanFactory则提供了单一类型、集合类型以及层次性等多种依赖查找方式。

## beanFactory.getBean是否线程安全

是线程安全的

# IOC依赖注入

## 注入方式

- 手动模式
  - xml资源模式
  - java注解模式 @Bean
  - API配置原信息：applicationContext.registerBeanDefinition(name, beanDefinitionBuilder.getBeanDefinition());
- 构造器注入 constructor
- setter注入的缺陷：setter注入是无序的，构造器注入是有序的
- 字段注入

### 接口回调注入

- Aware系列回调
  - BeanFactoryAware
  - ApplicationContextAware

日志打印为两个true，证明beanFactory和applicationContext是同一个

```java
private static BeanFactory beanFactory;

private static ApplicationContext context;

public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(AwareDependencyDemo.class);
    applicationContext.refresh();

    System.out.println(beanFactory == applicationContext.getBeanFactory());
    System.out.println(context == applicationContext);
    applicationContext.close();
}

@Override
public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
    this.beanFactory=beanFactory;
}

@Override
public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.context=applicationContext;
}
```

### 限定注入

- 使用注解@Qualifier限定

  - 通过Bean名称限定
  - 通过分组限定


因为使用了Qualifier，所以persons2只注入了对应的bean集合，而persons注入了所有bean

```java
public class QualifierDependencyDemo {

    @Autowired
    private List<Person> persons;
    @Autowired
    @Qualifier
    private List<Person> persons2;
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(QualifierDependencyDemo.class);
        applicationContext.refresh();

        QualifierDependencyDemo bean = applicationContext.getBean(QualifierDependencyDemo.class);
        //person person1 person2
        System.out.println(bean.persons);
        //person2, person3
        System.out.println(bean.persons2);
        applicationContext.close();
    }
    @Bean
    public SuperPerson superPerson() {
        return new SuperPerson();
    }
    @Bean
    public Person person1() {
        return new Person(1);
    }
    @Bean
    @Qualifier
    public Person person2() {
        return new Person(2);
    }
}
```

- 基于注解@Qualifier拓展限定
  - 自定义注解

定义一个注解，它标注了Qualifier

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier
public @interface GroupBean {
}
```

使用GroupBean注解

可以看到，GroupBean注解的只有对应的bean，Qualifier有Qualifier和GroupBean注解对应分组的bean

```java
public class QualifierDependencyDemo {

    @Autowired
    private List<Person> persons;

    @Autowired
    @Qualifier
    private List<Person> persons2;

    @Autowired
    @GroupBean
    private List<Person> persons3;
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(QualifierDependencyDemo.class);
        applicationContext.refresh();

        QualifierDependencyDemo bean = applicationContext.getBean(QualifierDependencyDemo.class);
        //person person1 person2
        System.out.println(bean.persons);
        //person2, person3
        System.out.println(bean.persons2);
        //person3
        System.out.println(bean.persons3);
        applicationContext.close();
    }

    @Bean
    @GroupBean
    public Person person3() {
        return new Person(3);
    }
}
```

## 延迟注入

- ObjectFactory
- ObjectProvider (推荐)

## 其他注入方式

定义一个实体类，供后面测试

```java
public class TestBean {

    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "TestBean{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```



- 在test类中获取

```java

```

### 注解方式

定义一个配置类，这个方法中的bean的id默认是方法名

```java
@Configuration
public class MainConfig {
    @Bean
    public TestBean testBean(){
        TestBean testBean = new TestBean();
        testBean.setUsername("lisi");
        return testBean;
    }
}
```

test类获取bean

```java
@Test
public void testBeanConfig(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
    TestBean testBean = (TestBean)applicationContext.getBean("testBean");
    System.out.println(testBean.toString());
}
```

如果想自定义bean的id

```java
@Bean("testBeanZdy")
public TestBean testBean(){
    TestBean testBean = new TestBean();
    testBean.setUsername("lisi");
    return testBean;
}
```



## 依赖处理过程

- 为什么字段注入是通过类查找依赖
  - 因为DependencyDescriptor继承了InjectionPoint
  - 在InjectionPoint中，getDeclaredType方法返回字段通过什么类型查找

### 注入普通了类分析

- 从DefaultListableBeanFactory#resolveDependency入手

- 定义一个启动类，注入一个Bean

```java
public class AnnotationDependencyResolveDemo {
    @Autowired
    private Person person;
}
```

- 看到对应的DependencyDescriptor，他描述了需要注入的对应的Bean信息
  - declaringClass:类信息，这里是demo类
  - superPerson：注入的字段名称
  - field：注入的字段信息

- 进入doResolveDependency方法

```java
//查看上级有没有被嵌套过
InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
try {
   //注入的字段类型，这里是superperson
   Class<?> type = descriptor.getDependencyType();
  //通过type(这里是Person)查到能够注入的bean
   Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
  

   String autowiredBeanName;
   Object instanceCandidate;
//判断是不是有多个bean
   if (matchingBeans.size() > 1) {
       //再次筛选符合条件的bean名称
      autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
      
      instanceCandidate = matchingBeans.get(autowiredBeanName);
   }
   else {
      //一个bean直接获取
      Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
      autowiredBeanName = entry.getKey();
      instanceCandidate = entry.getValue();
   }

   if (autowiredBeanNames != null) {
      autowiredBeanNames.add(autowiredBeanName);
   }
   if (instanceCandidate instanceof Class) {
      //调用beanFactory.getBean(beanName)获取bean
      instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
   }
   return result;
}
```

## 使用注解进行包扫描

将指定包下的所有标注了@Controller、@Service、@Repository，@Component扫入容器

```java
@Configuration
@ComponentScan(value = "com.xiao")
public class MainConfig {
    @Bean("testBeanZdy")
    public TestBean testBean(){
        TestBean testBean = new TestBean();
        testBean.setUsername("lisi");
        return testBean;
    }
}
```

排除controller注解，将其不纳入扫描范围

```java
@Configuration
@ComponentScan(value = "com.xiao",excludeFilters = {
        @ComponentScan.Filter(type= FilterType.ANNOTATION, classes = {Controller.class}),
})
```

只扫描controller组件

```java
@Configuration
@ComponentScan(value = "com.xiao",includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
}, useDefaultFilters = false)
```

## Autowire注入过程

注意AutowiredAnnotationBeanPostProcessor#postProcessProperties

和MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition进行元信息操作

**注入在postProcessProperties 方法执行，早于setter注入， 也早于@PostConstruct**

 ## 自定义注入

- 定义一与Autowired注解类似的注解

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InjectPerson {
}
```

- 注入内容

```java
@InjectPerson
private Person injectPerson;
```

- 生成注解处理post
  - AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME：在AnnotationConfigUtils定义了这个名字，如果这个名称已经产生，则不再重新注入AutowiredAnnotationBeanPostProcessor
  - static：如果是非静态的一个方法，当前bean产生依赖当前所在的类，定义为static能让这个bean独立于所在类

```java
@Bean(name = AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)
public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
    AutowiredAnnotationBeanPostProcessor beanPostProcessor = new AutowiredAnnotationBeanPostProcessor();
    beanPostProcessor.setAutowiredAnnotationType(InjectPerson.class);
    return beanPostProcessor;
}
```

- 我们发现这样的注入方式，只能处理inject注解，不能处理autowired注解
- 因为AnnotationConfigUtils的缘故，所以我们重新生产bean，并且，加入order，让其晚于AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME生成

```java
@Bean
@Order(Ordered.LOWEST_PRECEDENCE - 3)
public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
    AutowiredAnnotationBeanPostProcessor beanPostProcessor = new AutowiredAnnotationBeanPostProcessor();
    beanPostProcessor.setAutowiredAnnotationType(InjectPerson.class);
    return beanPostProcessor;
}
```

如此，InjectPerson就能注入进去对应的bean了

## 组件的过滤规则

FilterType.ANNOTATION：按照注解
FilterType.ASSIGNABLE_TYPE：按照给定的类型（如:指定某个接口的实现类，就只扫描这个接口的实现类）；
FilterType.ASPECTJ：使用ASPECTJ表达式
FilterType.REGEX：使用正则指定
FilterType.CUSTOM：使用自定义规则

自定义规则示例

```java
public class MyTestFilter implements TypeFilter {
    /**
     *
     * @param metadataReader 获取当前扫描类的信息
     * @param metadataReaderFactory 获取其他所有类的信息
     * @return
     * @throws IOException
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取当前类注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前类信息
        metadataReader.getClassMetadata();
        //获取当前类资源， 如类路径等
        metadataReader.getResource();
        //如果返回true，则意味着将当前类加入容器中
        return false;
    }
}
```

## 懒加载

第一次获取bean的时候加载bean的对象，默认时初始化的时候就把bean对象创建好了

```java
@Bean
@Lazy
public TestBean testBean(){
    System.out.println("创建对象bean、、、、");
    TestBean testBean = new TestBean();
    testBean.setUsername("lisi");
    return testBean;
}
```

## 条件装配（condition）

@condition注解，spring 4.0后产生，大量运用于spring boot中

比如，某两个bean，我们需要一个在Windows环境下注入，另一个需要在linux环境下注入

配置windows的condition，如果matches返回true，则注入bean

```java
public class WindowsCondition implements Condition {
    /**
     * 如果为false，被bean不生效
     * @param conditionContext
     * @param annotatedTypeMetadata
     * @return
     */
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //获取bean上下文的工厂
        ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
        //获取当前环境信息
        Environment environment = conditionContext.getEnvironment();
        //判断是否时windows环境
        if(environment.getProperty("os.name").contains("Windows")){
            return true;
        }
        return false;
    }
}
```

linux的condition，这里直接写了false

```java
public class LinuxConfition implements Condition {
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return false;
    }
}
```

config类的方法上加入对应的@Conditional， @Conditional可以加在类上，如果加在类上，则对整个congfig类中配置的bean都生效，Conditional配置时数组类型，可以配置多个

```java
@Bean
@Conditional({WindowsCondition.class})
public TestBean testBeanWindows(){
    TestBean testBean = new TestBean();
    testBean.setUsername("windows");
    return testBean;
}

@Bean
@Conditional({LinuxConfition.class})
public TestBean testBeanLinux(){
    TestBean testBean = new TestBean();
    testBean.setUsername("Linux");
    return testBean;
}
```

## @import导入组件

这里improt导入的组件，默认bean的id是全类名

如：com.xiao.entry.TestImport

```java
@Configuration
@ComponentScan(value = "com.xiao",includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
}, useDefaultFilters = false)
@Import({TestImport.class})
public class MainConfig 
```

```java
public class TestImport {
}
```

## ImportSelector

自定义逻辑返回需要的组件

写一个自己的selector，将要导入的组件全类名写入返回的数组中

```java
public class TestImportSelector implements ImportSelector {
    /**
     * AnnotationMetadata:当前标注@Import注解的类的所有注解信息
     * @param annotationMetadata
     * @return 导入到容器中的组件全类名
     */
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //这里不能返回null，要么就返回空数组
        //return new String[0];
        return new String[]{"com.xiao.entry.MySelectorBean"};
    }
}
```

配置类中导入

```java
@Import({TestImport.class, TestImportSelector.class})
public class MainConfig
```

打印可以看到对应的组件

mainConfig
testController
com.xiao.entry.TestImport
com.xiao.entry.MySelectorBean
testBean
testBeanWindows

## ImportBeanDefinitionRegistrar

手动的根据容器中条件注入bean

```java
public class MyImprotBeanDef implements ImportBeanDefinitionRegistrar {
    /**
     * 手动的注入bean信息
     * @param annotationMetadata
     * @param beanDefinitionRegistry 所有注入容器的bean信息
     */
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata,
                                        BeanDefinitionRegistry beanDefinitionRegistry) {
        //判断是否存在testBean
        boolean testBean = beanDefinitionRegistry.containsBeanDefinition("testBean");
        if(testBean){
            //如果存在则注入myDefBean
            RootBeanDefinition rootBeanDefinition 
                = new RootBeanDefinition(TestImport.class);
            beanDefinitionRegistry
                .registerBeanDefinition("myDefBean", rootBeanDefinition);
        }
    }
}
```

```java
@Import({TestImport.class, TestImportSelector.class, MyImprotBeanDef.class})
public class MainConfig
```

打印结果：

```console
mainConfig
testController
com.xiao.entry.TestImport
com.xiao.entry.MySelectorBean
testBean
testBeanWindows
myDefBean
```

## spring的工厂bean

```java
public class TestFactoryBean implements FactoryBean<TestBean> {
    //创建对象注入ioc容器中
    public TestBean getObject() throws Exception {
        return new TestBean();
    }

    //是否单例模式
    public boolean isSingleton() {
        return false;
    }

    public Class<?> getObjectType() {
        return TestBean.class;
    }
}
```

```java
@Bean
public TestFactoryBean testFactoryBean(){
    return new TestFactoryBean();
}
```

```java
@Test
public void testBeanConfig2(){

    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
    //TestBean testBean = (TestBean)applicationContext.getBean("testBean");
    System.out.println("容器已加载完");
    //TestBean testBean = (TestBean)applicationContext.getBean("testBean");
    TestBean testBean = (TestBean)applicationContext.getBean("testFactoryBean");
    System.out.println(testBean.getClass());
}
```

输出结果，可以看到虽然获取的是testFactoryBean，但其类型时testbean类型

容器已加载完
class com.xiao.entry.TestBean ...

## spring 的生命周期

### 第一种方式

在注入bean的时候，执行初始化方法，在销毁容器时，执行bean的销毁方法

如果时多实例的，只会执行初始化方法

```java
public class TestInit {
    public void init(){
        System.out.println("test init");
    }

    public void destroy(){
        System.out.println("test destroy");
    }
}
```

```java
@Bean(initMethod = "init", destroyMethod = "destroy")
public TestInit testInit(){
    return new TestInit();
}
```

### 第二种方式

实现InitializingBean，DisposableBean接口，实现对应方法，将类注入ioc容器中

```java
@Component
public class Cat implements InitializingBean,DisposableBean {
    //容器销毁时执行方法
    public void destroy() throws Exception {
        System.out.println("Cat.....destroy");
    }
    //加载完属性执行方法，也就是初始化
    public void afterPropertiesSet() throws Exception {
        System.out.println("Cat ... init ...");
    }
}
```

### 第三种方式 JSR250

利用jsr250注解

```java
@Component
public class TestJsr250 {
    //bean初始化完成，并且赋值完成调用
    @PostConstruct
    public void init(){
        System.out.println("TestJsr250 ... init");
    }
    //容器销毁bean时调用
    @PreDestroy
    public void destroy(){
        System.out.println("TestJsr250 ... destroy");
    }
}
```

### 所有bean初始化前后调用(后置处理器)

所有bean在调用初始化方法前后时都会调用下面的方法

这个在底层大量运用，如

bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor;

```java
/**
 * 所有bean初始化前后调用
 * Created by Administrator on 2019/6/29.
 */
@Component
public class TestBeanPropersecor implements BeanPostProcessor {
    /**
     * bean调用初始化方法之前调用
     * @param o bean对象
     * @param s bean id名
     * @return
     * @throws BeansException
     */
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("BeforeInitialization.. "+o.getClass() + " bean name："+s);
        //返回bean对象，这里可以根据一些操作，返回处理后的bean
        return o;
    }

    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("AfterInitialization.. "+o.getClass() + " bean name："+s);
        return o;
    }
}
```

## @value注入属性值

 @Value("张三"):直接赋值方式

 @Value("#{2+3}")：SPEL表达方式

 @Value("${}"):直接取配置文件属性值

```java
public class TestBean {

    @Value("张三")
    private String username;
    @Value("#{2+3}")
    private String password;
```

### 原理

- 在DefaultListableBeanFactory#doResolveDependency中

```java

String strVal = resolveEmbeddedValue((String) value);//获取@Value的value值
Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);
//获取解析的值
String strVal = resolveEmbeddedValue((String) value);
```



## 获取配置文件属性值

建立配置文件testBean.properties

testBean.realName=真实的张三

建立实体bean，使用${testBean.realName}获取配置文件信息

```java
public class TestBean {

    @Value("张三")
    private String username;
    @Value("#{2+3}")
    private String password;
    @Value("${testBean.realName}")
    private String realname;
```

在配置java类中，获取配置文件信息，不要手贱加空格哈，我就加了，哈哈哈哈。。。。

```java
@Configuration
@PropertySource(value = "classpath:/testBean.properties")
public class MainConfig2 {
    @Bean
    public TestBean testBean(){
       return  new TestBean();
    }
}
```

也可以通过applicationContext获取配置信息

打印：

TestBean{username='张三', password='5', realname='真实的张三'}
真实的张三

```java
@Test
public void testBeanConfig1(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
    //获取这个配置类中的容器中的所有bean
    TestBean testBean = (TestBean)applicationContext.getBean("testBean");
    System.out.println(testBean.toString());
    //还可以通过这种方式获取配置信息
    ConfigurableEnvironment environment = applicationContext.getEnvironment();
    String realname =  environment.getProperty("testBean.realName");
    System.out.println(realname);
    applicationContext.close();
}
```

## @Primary首选bean

@autowired注解注入bean时，默认的时先找同class类型的bean，如果有多个bean，则按照名称注入，此时可以指定@qualifile来注入，但如果不想这样写时，可以用@Primary方式，让这个产生的bean优先注入

```
@Bean
@Primary
public TestBean testBean(){
   return  new TestBean();
}
```

## @autowired

自动装配位置：构造函数、set方法上、参数上

在有参构造方法中，默认会寻找参数所注入的bean，如：这个时候，默认注入的就是testBean

```java
@Service
public class TestService {
    private TestBean testBean;

    public TestService(TestBean testBean) {
        this.testBean = testBean;
    }
```

在@bean中，也可以为参数默认注入bean，这个@autowired可以省略

```java
@Bean
@Primary
public TestBean testBean(@Autowired Cat cat){
   return  new TestBean();
}
```

## 注入spring底层组件（awar）

只需要实现awar下面的各类接口，就可以调用底层的组件

```java
@Component
public class Red implements ApplicationContextAware, BeanNameAware,EmbeddedValueResolverAware {

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("获取容器信息:"+applicationContext.getClass());
    }

    public void setBeanName(String s) {
        System.out.println("获取该类的beanname："+s);
    }

    public void setEmbeddedValueResolver(StringValueResolver stringValueResolver) {
        System.out.println("取出容器加载配置文件的属性值："+stringValueResolver.resolveStringValue("${testBean.realName}"));
    }
}
```

输出结果：

获取该类的beanname：red
取出容器加载配置文件的属性值：真实的张三
获取容器信息:class org.springframework.context.annotation.AnnotationConfigApplicationContext

## 生产、测试、开发环境切换

建立一个config，用@profile标注个个环境产生的bean（如果不标示，则标示默认产生）

```java
@Configuration
public class TestProfile {

    @Profile(value = "test")
    @Bean("testTestBean")
    public TestBean testBeanTest(){
        return new TestBean();
    }

    @Profile(value = "prod")
    @Bean("prodTestBean")
    public TestBean testBeanPro(){
        return new TestBean();
    }

    @Profile(value = "dev")
    @Bean("devTestBean")
    public TestBean testBeanDev(){
        return new TestBean();
    }
}
```

1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test

2、代码的方式切换产生对应的bean 

```java
@Test
public void testBeanConfig2(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    //设置激活的环境
    applicationContext.getEnvironment().setActiveProfiles("prod");
    //配置主配置类
    applicationContext.register(TestProfile.class);
    //启动刷新容器
    applicationContext.refresh();

    String[] beanNamesForType = applicationContext.getBeanNamesForType(TestBean.class);
    for(String beanName : beanNamesForType){
        System.out.println(beanName);
    }
    applicationContext.close();
}
```

# 依赖来源

依赖注入会比依赖来源多一项非spring容器管理对象

即：

```java
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```

这四个

BeanFactory不能通过autowire的方式注入，其他都可以

## 容器管理

元信息：是不是primary,lazy

| 来源                       | bean对象 | 生命周期管理 | 配置元信息 | 使用场景       |
| -------------------------- | -------- | ------------ | ---------- | -------------- |
| spring beanDefinition      | y        | Y            | Y          | 依赖注入，查找 |
| 单体对象(beanDefinition等) | y        | n            | n          | 依赖注入，查找 |
| Resolvable Dependency      | N        | N            | N          | 依赖注入       |

## BeanDefinition

- 注册中心接口：BeanDefinitionRegistry，它提供了一些正常的CURD的接口方法
- 它的实现：DefaultListableBeanFactory
- 通过DefaultListableBeanFactory#registerBeanDefinition(String beanName, BeanDefinition beanDefinition)方法注入
- 将其注入到对应的集合中

```java
//存储集合
this.beanDefinitionMap.put(beanName, beanDefinition);
//维护顺序
this.beanDefinitionNames.add(beanName);
```

- 最后通过BeanDefinition元信息创建bean

## 注入和查找来源是否相同

否，查找来源Spring BeanDefinition 以及单例对象，而注入来源还包括Resolvable Dependency 以及@Value 外部配置

# Bean 作用域

## singleton

- 主要是由BeanDefinition#isSingleton来进行元信息的判断
- singleton 查找和注入都是同一个同一个对象
- prototype 查找和注入 都是新生成的对象
- singleton 有init和destroy  
- prototype只有init

## request scop

- 每次返回前端的bean是新生成的
- 但是后端的bean是cglib提升的，单例的

## ApplicationScope

- API:ServletContextScope

## 自定义作用域

- 实现scope

```java
public class ThreadLocalScope implements Scope {

    public static final String SCOPE_NAME="thread-local";

    private final NamedThreadLocal<Map<String, Object>> threadLocal = new NamedThreadLocal("thread-local-scope") {
        @Override
        protected Object initialValue() {
            //没有获取到对象是兜底返回
            return new HashMap<>();
        }
    };

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> context = threadLocal.get();
        Object object = context.get(name);
        if(ObjectUtils.isEmpty(object)) {
            object = objectFactory.getObject();
            context.put(name, object);
        }
        return object;
    }

    @Override
    public Object remove(String name) {
        return threadLocal.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {

    }

    @Override
    public Object resolveContextualObject(String key) {
        return threadLocal.get().get(key);
    }

    @Override
    public String getConversationId() {
        return String.valueOf(Thread.currentThread().getId());
    }
}
```

- 注入scope

```java
public class ThreadLocalScopeDemo {

    @Bean
    @Scope(ThreadLocalScope.SCOPE_NAME)
    public Person person() {
        return new Person(String.valueOf(Thread.currentThread().getId()));
    }

    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(ThreadLocalScopeDemo.class);
        applicationContext.addBeanFactoryPostProcessor(beanFactory -> {
            beanFactory.registerScope(ThreadLocalScope.SCOPE_NAME, new ThreadLocalScope());
        });

        applicationContext.refresh();
        for(int i=0; i<3; i++) {
            new Thread(() -> {
                Person bean = applicationContext.getBean(Person.class);
                System.out.println(bean);
                Person bean1 = applicationContext.getBean(Person.class);
                System.out.println(bean1);
            }).start();
        }
        Thread.sleep(Integer.MAX_VALUE);
        applicationContext.close();
    }
}
```



# Bean 元信息配置

- BeanDefinition配置

## 面向资源

### xml配置方式

1. 在resources下建立bean.xml文件

```xml
<!--以配置文件的方式配置bean-->
<bean id="testBean" class="com.xiao.entry.TestBean" >
    <property name="username" value="xiao"></property>
    <property name="password" value="123456"></property>
</bean>
```

2. 注入bean和获取bean

```java
@Test
public void testBeanXml(){
    //直接从bean文件获取bean
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
    TestBean testBean = (TestBean) applicationContext.getBean("testBean");
    System.out.println(testBean.toString());
}
```

### properties 

- 遵循的格式可以参考PropertiesBeanDefinitionReader
- 如employee.(class)=MyClass  表示指定Bean的class

- 在resource下定义数据

```properties
person.(class)=com.xiao.pojo.Person
person.age=1
person.name=老肖
```

- 定义加载类

```java
public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        PropertiesBeanDefinitionReader beanDefinitionReader = new PropertiesBeanDefinitionReader(beanFactory);
        String location = "application.properties";
        EncodedResource encodedResource = new EncodedResource(new ClassPathResource(location), "UTF-8");
        int i = beanDefinitionReader.loadBeanDefinitions(encodedResource);
        System.out.println("加载bean数量: " + i);

        Person bean = beanFactory.getBean(Person.class);
        System.out.println(bean);
    }
```

## Bean Class 加载

- AbstractBeanFactory#doGetBean 进入
- 在获取完beanDefinition后

```java
final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
```

- 调用DefaultSingletonBeanRegistry#getSingleton方法

- 如果没有找到bean，则执行方法进行创建bean

```java
sharedInstance = getSingleton(beanName, () -> {
   try {
      return createBean(beanName, mbd, args);
   }
});
```

- 进入AbstractAutowireCapableBeanFactory#createBean方法创建bean

```java
//解析bean的class（利用java的classload）
Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
```

## Bean实例化前

- 每实例化bean都会拿出所有的前置处理进行调用
- InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation  
- bean实例化前进行加载，如果返回对象不为空，则直接使用当前对象
- 用处：如需要实现自己的远程bean等一些

- 实例：

```java
public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(InstantiationBeforeProcessor.class);
        applicationContext.addBeanFactoryPostProcessor(beanFactory -> {
            beanFactory.addBeanPostProcessor(new MyInstantiationBeanProcessor());
        });
        applicationContext.refresh();

        applicationContext.close();
    }

    static class MyInstantiationBeanProcessor implements InstantiationAwareBeanPostProcessor {
        @Override
        public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
            System.out.println(beanClass);
            return null;
        }
    }
```

## 实例化

### 传统

- 从AbstractAutowireCapableBeanFactory#doCreateBean
- 进入AbstractAutowireCapableBeanFactory#createBeanInstance

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {


   Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
   if (instanceSupplier != null) {
      return obtainFromSupplier(instanceSupplier, beanName);
   }

   if (mbd.getFactoryMethodName() != null) {
       //工厂方法进行实例化
      return instantiateUsingFactoryMethod(beanName, mbd, args);
   }


   // Candidate constructors for autowiring?
   Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
   if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
         mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
       //带参构造函数初始化
      return autowireConstructor(beanName, mbd, ctors, args);
   }

   ctors = mbd.getPreferredConstructors();
   if (ctors != null) {
      return autowireConstructor(beanName, mbd, ctors, null);
   }
	//默认实例化，无构造参数实例化
   return instantiateBean(beanName, mbd);
}
```

- AbstractAutowireCapableBeanFactory#instantiateBean

```java
//使用策略的方式进行调用创建方法
beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
```

- SimpleInstantiationStrategy#instantiate
- 最后调用BeanUtils.instantiateClass(constructorToUse);生成bean

### 构造器注入

- 按照类型注入
- resolveDependency

## 实例化后

- InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
- 已经实例化bean了，但是没有注入属性
- 如果我们不想走传统复制方式，则可以采用这种方式赋值

- 在AbstractAutowireCapableBeanFactory#doCreateBean中

```java
Object exposedObject = bean;
try {
    //实例化以后注入属性
   populateBean(beanName, mbd, instanceWrapper);
   exposedObject = initializeBean(beanName, exposedObject, mbd);
}
```

- 使用方式

```java
static class MyInstantiationBeanProcessor implements InstantiationAwareBeanPostProcessor {

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(ObjectUtils.nullSafeEquals(beanName, "person")) {
            Person person = Person.class.cast(bean);
            person.setAge(2);
            return true;
        }
        return false;
    }
}
```

## 属性赋值前

- InstantiationAwareBeanPostProcessor#postProcessProperties

## Bean Aware

- BeanNameAware
- BeanClassLoaderAware
- BeanFactoryAware

## Bean 初始化前

- 这个时候已经完成了Bean实例化、Bean属性赋值、Aware接口回调

- BeanPostProcessor#postProcessBeforeInitialization
- 从源码可以看出，如果返回的不为空，则会直接当成bean（可以返回代理对象）使用

```java
for (BeanPostProcessor processor : getBeanPostProcessors()) {
   Object current = processor.postProcessBeforeInitialization(result, beanName);
   if (current == null) {
      return result;
   }
   result = current;
}
```

- 这个执行在bean初始化方法里

```java
Object exposedObject = bean;
try {
   populateBean(beanName, mbd, instanceWrapper);
   //执行初始化
   exposedObject = initializeBean(beanName, exposedObject, mbd);
}
```



## 初始化阶段

- 

## 初始化后

- BeanPostProcessor#postProcessAfterInitialization

## 初始化完成

- SmartInitializingSingleton#afterSingletonsInstantiated
- 通常在spring applicationcontext调用
- 当前容器所有的beandefinition已经完全初始化后调用

## Bean 销毁前阶段

- DestructionAwareBeanPostProcessor#postProcessBeforeDestruction

## Aware相关执行

- 通过org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces方法的顺序进行执行

# Bean配置元信息

- 配置元信息   BeanDefinition
- 属性元信息 propertyValues
  - 他是个集合
- 外部化元信息  propertySource
- Profile元信息  @Profile
  - 如生产测试环境等的区分
  - 在Environment#getDefaultProfiles中体现

## XML配置元信息

- 具体可以参考BeanDefinitionParserDelegate
- 里面的默认值属性
- 实现类为：XmlBeanDefinitionReader
- 加载顺序XmlBeanDefinitionReader#loadBeanDefinitions->doLoadBeanDefinitions->registerBeanDefinitions->DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions(解析)

### 生效配置

- 在META-INF/spring.handlers下，会有对应的命名空间对应的handler
- META-INF/spring.schemas下有对应的xsd映射关系

## properties配置元信息

- 实现类：PropertiesBeanDefinitionReader

## 注解的方式

- 在ClassPathScanningCandidateComponentProvider#registerDefaultFilters中
- 注册相关注解

```java
protected void registerDefaultFilters() {
   this.includeFilters.add(new AnnotationTypeFilter(Component.class));
```

- 基于AnnotatedBeanDefinitionReader实现

```java
public class AnnotatedBeanDefinitionReader {
	//bean名称的generator
   private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;
//解析元信息的相关信息，如代理类等信息
   private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();
    //是否注册bean（条件）
    private ConditionEvaluator conditionEvaluator;
```

- 在AnnotatedBeanDefinitionReader#doRegisterBean进行注册

### 转配注解

- @ImportResource   替换xml的<import>， 可以直接导入xml 的配置文件
- @Import  导入 Configruation class
- @ ComponentScan  扫描指定包

### 配置属性

- @PropertySource   利用java8特性，可以导入多个property文件

```java
@PropertySource("某配置1.properties")
@PropertySource("某配置2.properties")
public class InstantiationBeforeProcessor {
```

- @PropertySources  PropertySource   的集合

## 扩展Spring XML文件

- 编写xml schema 文件（定义xml的结构）
  - 在resource下建立com/xiao/in/spring/xml/persons.xsd文件
  - 可以仿照spring-beans.xsd
  - id这个属性一定要配置

```scheme
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<xsd:schema xmlns="http://www.xiao.org/schema/persons"
            xmlns:xsd="http://www.w3.org/2001/XMLSchema"           targetNamespace="http://www.xiao.org/schema/persons">
    <xsd:import namespace="http://www.w3.org/XML/1998/namespace"/>
    <!--定义复杂类型（person）-->
    <xsd:complexType name="Person">
    	<xsd:attribute name="id" type="xsd:string" 			         use="required"></xsd:attribute>
        <!-- 必须填写-->
        <xsd:attribute name="name" type="xsd:string" use="required"></xsd:attribute>
        <xsd:attribute name="age" type="xsd:integer" ></xsd:attribute>
    </xsd:complexType>

    <!--定义person 元素-->
    <xsd:element name="person" type="Person"></xsd:element>
</xsd:schema>
```

- 定义xml引用
  - 类似spring-bean.xml的文件
  - 在META-INF下定义文件person-context.xml
  - xmlns:persons要和persons.xsd定义一致

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:persons="http://www.xiao.org/schema/persons"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.xiao.org/schema/persons
        http://www.xiao.org/schema/persons.xsd
">
   <persons:person id="person" name="老肖" age="1"></persons:person>
</beans>
```

- 自定义namespaceHandlers实现（命名空间绑定）
- 自定义BeanDefinition的解析

```java
public class PersonNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        registerBeanDefinitionParser("person", new PersonBeanDefinitionParser());
    }

    private class PersonBeanDefinitionParser extends AbstractSingleBeanDefinitionParser {
        @Override
        protected Class<?> getBeanClass(Element element) {
            return Person.class;
        }

        @Override
        protected void doParse(Element element, BeanDefinitionBuilder builder) {
            this.setAttribute("name", element.getAttribute("name"), builder);
            this.setAttribute("age", element.getAttribute("age"), builder);
        }

        private void setAttribute(String name, String value, BeanDefinitionBuilder builder){
            Optional.ofNullable(value).ifPresent(v -> builder.addPropertyValue(name, v));
        }
    }
}
```

- 定义handler映射
  - 配置spring.hanlders文件

```handlers
http\://www.xiao.org/schema/persons=com.xiao.in.spring.xml.PersonNamespaceHandler
```

- 注册XML扩展
- 命名空间映射
  - 定义spring.schemas文件

```java
http\://www.xiao.org/schema/persons.xsd=com/xiao/in/spring/xml/persons.xsd
```

- 解析测试

```java
public static void main(String[] args) {
    DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
    XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    xmlBeanDefinitionReader.loadBeanDefinitions("META-INF/person-context.xml");
    Person bean = beanFactory.getBean(Person.class);
    System.out.println(bean);
}
```

## 扩展xml原理

AbstractApplicationContext#obtainFreshBeanFactory

->AbstractRefreshableApplicationContext#refreshBeanFactory

->AbstractXmlApplicationContext#loadBeanDefinitions

->XmlBeanDefinitionReader#doLoadBeanDefinitions

->DefaultBeanDefinitionDocumentReader#parseBeanDefinitions

## YAML资源装载


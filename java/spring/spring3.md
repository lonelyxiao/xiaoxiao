# 后置处理器原理

> BeanPostProcessor

- 主要在bean的实例化前后执行

```java
public interface BeanPostProcessor {
   @Nullable
   default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }
   @Nullable
   default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

}
```

> > 注册阶段

在refresh()的registerBeanPostProcessors(beanFactory);中注册

> InstantiationAwareBeanPostProcessor

- InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation在bean的实例化前执行，如果他返回了bean，则不再创建bean

# Bean配置元信息

## 基于XML 资源装载Spring Bean配置元信息

| xml元素 | 使用场景                                  |
| ------- | ----------------------------------------- |
| beans   | 单XML资源下的多个Spring Beans配置         |
| bean    | 单个 Spring Bean定义(BeanDefinition)配置  |
| alias   | 为Spring Bean定义(BeanDefinition)映射别名 |
| import  | 加载外部 Spring XML配置资源               |

> 底层主要是通过XmlBeanDefinitionReader来进行加载的

-  XmlBeanDefinitionReader有两种加载方式，一个是通过DTD,一个是通过XSD方式

```java
/**
 * Indicates that DTD validation should be used.
 */
public static final int VALIDATION_DTD = XmlValidationModeDetector.VALIDATION_DTD;

/**
 * Indicates that XSD validation should be used.
 */
public static final int VALIDATION_XSD = XmlValidationModeDetector.VALIDATION_XSD;
```

- beandfiniton是通过bean的定义来进行加载的，如：user 的bean先定义，则先解析到beandefinition（**但是我们bean的使用又不是有序的**）

> > 加载bean的流程

1. refresh()方法
2. 调用obtainFreshBeanFactory()刷新bean工厂

3. 在AbstractXmlApplicationContext#loadBeanDefinitions方法中获取对应的xml解析器

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory){
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
```

4. 调用AbstractBeanDefinitionReader#loadBeanDefinitions进行解析xml
5. 调用XmlBeanDefinitionReader#doLoadBeanDefinitions进行xml文件的解析和beandefintion的设置
6. 调用XmlBeanDefinitionReader#registerBeanDefinitions进行beandefintion的设置

```java
public int registerBeanDefinitions(Document doc, Resource resource){
    //创建一个beandefintion与xml的关系连接的解读器
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   int countBefore = getRegistry().getBeanDefinitionCount();
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

7. 最终调用DefaultBeanDefinitionDocumentReader#processBeanDefinition方法，将解析好的bean register到beandefintion中

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
   BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   if (bdHolder != null) {
       //获取到的bean的别名，id等一些信息
      bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
      try {
         BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
      }
      getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
   }
}
```

> 总结

通过xml的资源，封装成BeanDefinitionHolder然后解析成beandefinition

## 注解方式装载元信息

> 核心类AnnotatedBeanDefinitionReader

```java
private final BeanDefinitionRegistry registry;

private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;
//Bean 范围解析
private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();
//条件评估，判断是否装载元信息
//如condition的条件是不是成立
private ConditionEvaluator conditionEvaluator;
```

> ConfigurationClassParser#doProcessConfigurationClass

当register之后，进入refresh阶段

在invokeBeanFactoryPostProcessors(beanFactory);，会将register注入的bean进行一个注解的包扫描

```java
// The config class is annotated with @ComponentScan -> perform the scan immediately
Set<BeanDefinitionHolder> scannedBeanDefinitions =
      this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
```



# 面试题

## 循环依赖

官网说明：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependency-resolution

![image-20210629165813136](https://gitee.com/xiaojihao/pubImage/raw/master/image/spring/20210629165813.png)

### 解决方案

- 构造器的注入方式没办法解决循环依赖问题
- 将scope设置@Scope("prototype")原型没办法解决循环依赖

- 使用set方式注入，可以解决循环依赖的问题



循环依赖的异常：

```tex
Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException
```

### 解决原理

- 在org.springframework.beans.factory.support.DefaultSingletonBeanRegistry中，有三个map，分别是三个缓存

```java
//一级缓存:存放的是已经初始化好了的Bean
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

//三级缓存:
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

//二级缓存: 存放的是实例化了，但是未初始化的Bean
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

- 步骤

1. A创建过程中需要B，于是A将自己放到三级缓里面，去实例化B
2. B实例化的时候发现需要A，于是B先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了A然后把三级缓存里面的这个A放到二级缓存里面，并删除三级缓存里面的A
3. B顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中状态,然后回来接着创建A，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，并将A自己放到一级缓存里面。

> ObjectFactory使用来干嘛的

在AbstractBeanFactory#doGetBean方法中，

创建bean的代码，可以看大getSingleton(String beanName, ObjectFactory<?> singletonFactory)的ObjectFactory实际上就是一个创建bean的方法

如果A发现需要B，B还没创建，此时，A是先不实例化的，B创建的时候，将A调用objectFactory,然后A放入了二级缓存，这里相当于一个递归的调用

```java
if (mbd.isSingleton()) {
   sharedInstance = getSingleton(beanName, () -> {
      try {
         return createBean(beanName, mbd, args);
      }
      catch (BeansException ex) {
         destroySingleton(beanName);
         throw ex;
      }
   });
   bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

如果没有三级缓存可不可以？

不可以，实现下，如果A直接实例化，放入二级缓存后，B去取A，那么将A注入后，B怎么让A调用createBean的方法呢？

如果没有二级缓存可不可以？

不可以，因为循环依赖是单例的，如果我们再次调用create，就会产生新的bean

## Bean的实例化流程

1. 实例化DefaultListableBeanFactory#preInstantiateSingletons方法中，

```java
//将所有的bean定义进行实例化操作
for (String beanName : beanNames) {
}
```

2. 进入AbstractBeanFactory#doGetBean方法

```java
//首先去一级缓存获取bean，如果没有判断其是不是正在创建，如果没有，则返回null
Object sharedInstance = getSingleton(beanName);
```

3. 在AbstractAutowireCapableBeanFactory#doCreateBean方法中

```java
//如果支持三级缓存，则将当前的bean，和objectfactory函数存入三级缓存
if (earlySingletonExposure) {
    //如果bean不在一级缓存，则存入三级缓存
    //如果三级缓存存在，则调用getEarlyBeanReference方法，获取未赋值的bean，设置进入二级缓存
   addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

4. 开始设置属性populateBean，如果我们通过**注解的方式**注入，则会调用**AutowiredAnnotationBeanPostProcessor**#postProcessProperties方式进行注入

然后再在DependencyDescriptor#resolveCandidate调用beanFactory.getBean(beanName)来获取依赖的bean

这个时候，调用getSingleton（）方法，去获取三级缓存的bean

**注意：此时，a的实例化方法还在调用中（如果是循环依赖），b的创建只是通过a的方法调用geBean的时候创建的**

# @Resource和@Autowire的区别

@Resource和@Autowired都可以用来装配bean，都可以用于字段或setter方法。
@Autowired默认按类型装配，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false。
@Resource默认按名称装配，当找不到与名称匹配的bean时才按照类型进行装配。名称可以通过name属性指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，当注解写在setter方法上时，默认取属性名进行装配。

# 几个不常用的后置处理器

## Bean 初始化后置处理器

>  BeanPostProcessor

bean在初始化前后调用方法

## Bean实例化后置处理

>  InstantiationAwareBeanPostProcessor

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

   @Nullable
   default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
      return null;
   }

   default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
      return true;
   }

   @Nullable
   default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
         throws BeansException {

      return null;
   }
}
```

## Bean工厂后置处理

>  BeanFactoryPostProcessor

bean 定义已经加载，还没创建对象的时候调用

我们可以用beanfactory进行一些操作：获取bean的定义名称

## Bean定义后置处理

>  BeanDefinitionRegistryPostProcessor

可以注入额外的bean组件

## 容器单实例bean创建后

>  SmartInitializingSingleton

容器单实例bean初始化后，执行

# Spring 资源管理

## Spring 内建Resource

| 类                 | 描述                |
| ------------------ | ------------------- |
| UrlResource        |                     |
| ClassPathResource  | 类路径  classpath:/ |
| FileSystemResource |                     |
| EncodedResource    | 带编码的resource    |

## 资源加载器

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/java/spring/20210218145156.png)

## 通配路径资源加载

- ResourcePatternResolver

```java
public static void main(String[] args) throws IOException {
    String currentPath = "/"+ System.getProperty("user.dir")+"/stu-spring/src/main/java/com/xiao/in/spring/resource/";
    String localPath = currentPath+"*.java";

    PathMatchingResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver(new FileSystemResourceLoader());
    Resource[] resources = patternResolver.getResources(localPath);
    Arrays.stream(resources).map(ResourcePatternResolverDemo::getContent).forEach(System.out::println);
}

public static String getContent(Resource resource) {
    EncodedResource encodedResource = new EncodedResource(resource, "UTF-8");
    try (Reader reader = encodedResource.getReader()){
        return IoUtil.read(reader);
    } catch (Exception e) {
    }
    return null;
}
```

## 注解的方式加载

```java
@Value("classpath:/application.properties")
private Resource resource;

@Value("classpath*:/META-INF/spring.*")
private Resource[] resources;

@PostConstruct
public void init() {
    System.out.println(resource.getFilename());
    System.out.println("=========");
    Arrays.stream(resources).map(Resource::getFilename).forEach(System.out::println);
 }

public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(InjectResourceDemo.class);
    applicationContext.refresh();
    applicationContext.close();
}
```

# 国际化

- 核心接口
  - MessageSource
- 国际化层次性接口HierarchicalMessageSource

## ResourceBundle核心特性

- Key value 设计
- 层次性设计
- 缓存设计
  - 一旦加载就缓存起来
  - ResourceBundle#getBundleImpl可以看出，他是先从缓存中取数据的
- 字符编码控制
  - java.util.ResourceBundle.Control#Control
  - 1.6开始支持

## 在ApplicationContext中初始化

- 通AbstractApplicationContext#initMessageSource方法初始化 

- refresh中调用

## Spring Boot使用

- MessageSourceAutoConfiguration#messageSource
- 注入messageSource的Bean

# Spring 校验

## Error文案

- FieldError是ObjectError子类，他多了关联的哪个字段
- reject:收集错误文案（如某个对象为空）
- rejectValue: 收集对象字段的错误（如某个字段 为空）

## Validate提升

- 当想创建基于spring validated的bean时，使用**LocalValidatorFactoryBean**创建
- 还有一个基于方法的AOP拦截
- MethodValidationPostProcessor， 他是基于Validated注解来拦截的

### 示例

- 引入jar包

```xml
<dependency>
  <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

- 主要是为了引入ELManager处理Spring 的el

```xml
<dependency>
    <groupId>org.mortbay.jasper</groupId>
    <artifactId>apache-el</artifactId>
</dependency>
```

### 检测jar包是否正常

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(BeanValidatedDemo.class);
    applicationContext.refresh();
    Validator bean = applicationContext.getBean(Validator.class);
    System.out.println(bean);
    applicationContext.close();
}

@Bean
static LocalValidatorFactoryBean validated() {
    return new LocalValidatorFactoryBean();
}
```

### 正式使用

- 定义方法级别的处理

```java
@Bean
static LocalValidatorFactoryBean validated() {
    LocalValidatorFactoryBean validator = new LocalValidatorFactoryBean();
    return validator;
}

@Bean
static MethodValidationPostProcessor methodValidator(Validator validator) {
    MethodValidationPostProcessor methodValidation = new MethodValidationPostProcessor();
    methodValidation.setValidator(validator);
    return methodValidation;
}
```

- 定义处理的pojo，NotNull标识不能为空

```java
@Setter
@Getter
@ToString
static class User {
    @NotNull
    private String name;

    private Integer id;
}
```

- 定义需要拦截的处理类，Validated表示这个类会生成代理类

```
@Component
@Validated
static class  UserProcess {
    public void process(@Valid User user) {
        System.out.println(user);
    }
}
```

- 调用

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(BeanValidatedDemo.class);
    applicationContext.refresh();
    UserProcess bean = applicationContext.getBean(UserProcess.class);
    bean.process(new User());
    applicationContext.close();
}
```

# 数据绑定

## DataBinder

- DataBinder 绑定方法
  - bind(PropertyValues pvs)
  - 通过PropertyValues的key-value与bean的属性映射

- 数据来源
  - BeanDefinition(xml格式的)
- 通过BeanDefinition#MutablePropertyValues.getPropertyValues();方法获取PropertyValues 值
- DataBinder 会默认忽略bean实体中不存在的属性
- 如果是嵌套属性，properties可以是user.name-value的方式，进行嵌套

### 绑定参数控制

```java
//是否忽略未知字段
private boolean ignoreUnknownFields = true;
//是否忽略非法字段
private boolean ignoreInvalidFields = false;
//是否增加前台路径，如user.name
private boolean autoGrowNestedPaths = true;

//绑定字段白名单
private String[] allowedFields;

//绑定字段黑名单
private String[] disallowedFields;

//必须绑定
private String[] requiredFields;
```

# 类型转换

## 实现

- 基于javaBeans接口实现
  - 基于java.beans.PropertyEditor拓展
- Spring 3.0+ 通用类型转换实现

## JavaBeans类型转换

- 职责
  - 将String 类型转为目标类型
- 扩展原理
  - Spring框架将文本内容传递到PropertyEditor 实现的setAsText(String)方法
  - PropertyEditort#setAsText(String)方法实现将String类型转化为目标类型的对象
  - 将目标类型的对象传入PropertyEditor#setValue(Object)方法暂存
  - Spring框架将通过PropertyEditortgetValue()获取类型转换后的对象
- 示例

```java
public static void main(String[] args) {
    StringToPropertyEditor editor = new StringToPropertyEditor();
    editor.setAsText("name=老肖");
    //最终会输出Property对象数据
    System.out.println(editor.getValue());
}

static class StringToPropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) throws java.lang.IllegalArgumentException {
        //将String 类型转为properties
        Properties properties = new Properties();
        try {
            properties.load(new StringReader(text));
        } catch (IOException e) {
            e.printStackTrace();
        }
        //暂存
        setValue(properties);
    }
}
```

## 通用类型转换

- 类型转换接口：org.springframework.core.convert.converter.Converter
  - 这个接口通过泛型来进行约束
- 通用类型转换接口：org.springframework.core.convert.converter.GenericConverter
  - 这个接口应用范围更广，使用TypeDescriptor类进行描述目标类型等

## GenericConverter

- 适合复杂类型转换，如集合，数组
- 可以转换的类型：
  - Set<ConvertiblePair>getConvertibleTypes();
- 优化
  - 融合了条件的接口

```java
interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter 
```

# 泛型处理

## 泛型辅助类

- 核心api GenericTypeResolver

```java
//类型相关方法
resolveReturnType(Method method, Class<?> clazz)
//泛型参数类型相关
resolveReturnTypeArgument(Method method, Class<?> genericIfc)
```

- 代码示例

```java
public class TypeResolverDemo {
    public static void main(String[] args) throws Exception {
        disableReturnGenericInfo(TypeResolverDemo.class, String.class, "getString");

        disableReturnGenericInfo(TypeResolverDemo.class, List.class, "getList");

        disableReturnGenericInfo(TypeResolverDemo.class, List.class, "getStringList");
    }

    public static void disableReturnGenericInfo(Class<?> containClass, Class typeClass,  String methodName, Class... argumentTypes) throws Exception {
        Method method = containClass.getMethod(methodName, argumentTypes);
        //获取常规方法返回的类型
        Class<?> returnType = GenericTypeResolver.resolveReturnType(method, containClass);
        System.out.println("方法返回["+methodName+"] 返回类型:"+returnType);

        //获取泛型方法返回(如果泛型未指定，则返回为空)
        Class<?> typeArgument = GenericTypeResolver.resolveReturnTypeArgument(method, typeClass);
        System.out.println("方法返回["+methodName+"] 返回类型:"+typeArgument);
    }

    public static String getString() {
        return null;
    }

    public static <E> List<E> getList() {
        return null;
    }

    public static List<String> getStringList() {
        return null;
    }
}
```

## 集合类型辅助

- 使用ResolvableType类
- 工厂方法： for*方法
- 转换方法： as*方法
- 处理方法: resolve*方法

## MethodParameter



# Spring 注解

## Spring模式注解

- @ComponentScan
  - ComponentScanAnnotationParser#parse(AnnotationAttributes componentScan, final String declaringClass)进行解析这个注解
  - 最后调用scanner.doScan(StringUtils.toStringArray(basePackages))来返回BeanDefinitionHolder集合
  - 在ClassPathBeanDefinitionScanner#doScan中，会去调用Set<BeanDefinition> candidates = findCandidateComponents(basePackage);方法，找出component注解标注，或者元标注的类的beandefinition

## Spring 注解别名

- 显性别名

如：ComponentScan的

```
@AliasFor("basePackages")
String[] value() default {};
```

在代码中既可以用basePackages，也可以同value

- 隐性别名

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@ComponentScan
public @interface MyComponentScan {

    //使用MyComponentScan就可以直接使用myScan替换basePackages了
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] myScan() default {} ;
}
```

- 隐性覆盖
  - 如果注解出现于元标注的注解同名的字段，则它的内容会覆盖它

# Spring 条件

- 主要ConditionEvaluator#shouldSkip方法进行判断
- 其中AnnotationAwareOrderComparator.sort(conditions);进行排序，将优先级高的取出来

# Spring AOP

## aop功能的测试

1 导入依赖包

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```

2 建立一个aop需要拦截的类

```java
public class AopClass {
    public String aopMethod(String name){
        return "进过aop method 方法处理："+name;
    }
}
```

3 定义日期切面方法

通知方法：
 * 前置通知(@Before)：logStart：在目标方法(div)运行之前运行
 * 后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
 * 返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
 * 异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
 * 环绕通知(@Around)：可以在方法之前、之后、发生异常时执行，手动推进目标方法运行（joinPoint.procced()）

给切面类的目标方法标注何时何地运行（通知注解:@Aspect）

```java
@Aspect
public class LogAspects {
    // * 表示所有方法， .. 表示多个参数
    @Pointcut("execution(public String com.xiao.aop.AopClass.*(..)) ")
    public void pointCut(){
    }
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(joinPoint.getSignature().getName()+"方法开始切入之前：参数："+args);
    }
    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }
    @AfterReturning(value = "pointCut()", returning ="result")
    public void logReturn(JoinPoint joinPoint, Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }
    @AfterThrowing(value = "pointCut()", throwing = "execution")
    public void logException(JoinPoint joinPoint, Exception execution){
        System.out.println(""+joinPoint.getSignature().getName()+"异常返回。。。@AfterThrowing:运行结果：{"+execution+"}");
    }
}
```

配置配置类

```java
@EnableAspectJAutoProxy
@Configuration
public class MainConfigAop {
    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
    @Bean
    public AopClass aopClass(){
        return new AopClass();
    }
}
```

调用：

```java
@Test
public void testBeanConfig3(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigAop.class);
    //获取这个配置类中的容器中的所有bean
    AopClass aopClass = (AopClass)applicationContext.getBean("aopClass");
    aopClass.aopMethod("老小");
    applicationContext.close();
}
```

输出：

aopMethod方法开始切入之前：参数：[Ljava.lang.Object;@1465398
进入方法
aopMethod结束。。。@After
aopMethod正常返回。。。@AfterReturning:运行结果：{进过aop method 方法处理：老小}

### 环绕通知

```java
public class SurroundMethod implements MethodInterceptor{
    public Object invoke(MethodInvocation invocation) {
        Object result = null;
        try {
            System.out.println("环绕通知里面的【前置通知】。。。");
            result = invocation.proceed();  //这里相当于执行目标方法 如果不写目标方法就不会执行
            // result是目标方法的返回值
            System.out.println("环绕通知里面的【后置通知】...");
        } catch (Throwable e) {
            System.out.println("这里是执行环绕通知里面的【异常通知】。。。");
            e.printStackTrace();
        } finally{
　　　　　　　System.out.println("这里是执行环绕通知里面的【最终通知】");
　　　　　}
        return result;
        //也可以返回其他  return “123”;  那么目标方法的返回值就是 "123"
    } 
}
```



## 总结

1 @Aspect 标示切面类，在切面方法中标示对应的处理时机

2 将切面类和被切类注入容器中

3 在配置类启动切面注解@EnableAspectJAutoProxy

## aop原理

总体概括：@EnableAspectJAutoProxy注解给容器创建和注册AnnotationAwareAspectJAutoProxyCreator的bean（后置处理器，意味着以后任何组件创建时，都要执行这个后置处理器方法）

在@EnableAspectJAutoProxy注解中，有一个@Import(AspectJAutoProxyRegistrar.class)注解

它导入了AspectJAutoProxyRegistrar组件

AspectJAutoProxyRegistrar注入了org.springframework.aop.config.internalAutoProxyCreator（AnnotationAwareAspectJAutoProxyCreator类） 的bean（自动代理处理器）

AnnotationAwareAspectJAutoProxyCreator：父类一层层的看的它实现了SmartInstantiationAwareBeanPostProcessor后置处理器，BeanFactoryAware自动装配BeanFactory

后置处理器创建过程：

1 传入配置类，创建ioc容器

2 注册配置类，调用refresh();刷新容器

3 进入refresh方法

```java
// Register bean processors that intercept bean creation.
registerBeanPostProcessors(beanFactory);
```

注册后置处理器方法

---

1) 进入registerBeanPostProcessors方法，先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor

```java
String[] postProcessorNames =
 beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
```



2) 优先注册实现了PriorityOrdered接口的BeanPostProcessor

3) 再给容器中注册实现了Ordered接口的BeanPostProcessor

4) 注册没实现优先级接口的BeanPostProcessor

​		注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中

在注册processor里面循环方法中getBean这个方法进入可以看到，所以这个时候已经穿件BeanPostProcessor的bean

```java
BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
```

```java
@Override
public Object getBean(String name) throws BeansException {
   return doGetBean(name, null, null, false);
}
```

doGetBean先尝试获取单实例bean

```java
if (mbd.isSingleton()) {
  sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
						@Override
						public Object getObject() throws BeansException {
							try {
								return createBean(beanName, mbd, args);
							}
							catch (BeansException ex) {
								destroySingleton(beanName);
								throw ex;
							}
						}
					})
```

在getSingleton方法中，这个object就是new ObjectFactory中的方法，它去创建了一个bean

```java
singletonObject = singletonFactory.getObject();
```

**如何创建BeanPostProcessor**

进入createBean的方法中

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
   if (logger.isDebugEnabled()) {
      logger.debug("Creating instance of bean '" + beanName + "'");
   }
```

进入doCreateBean方法，看到instanceWrapper = createBeanInstance(beanName, mbd, args);

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
      throws BeanCreationException {

   // Instantiate the bean.
   BeanWrapper instanceWrapper = null;
   if (mbd.isSingleton()) {
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
   }
   if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }
   populateBean(beanName, mbd, instanceWrapper);
```

**populateBean(beanName, mbd, instanceWrapper)**给bean初始化

初始化流程：

这在initializeBean方法中看到invokeAwareMethods方法，这个方法是用来awar接口的方法回调

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged(new PrivilegedAction<Object>() {
         @Override
         public Object run() {
            invokeAwareMethods(beanName, bean);
            return null;
         }
      }, getAccessControlContext());
   }
   else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}
```

我们可以看到，applyBeanPostProcessorsBeforeInitialization方法，这个后置处理器处理方法，调用所有的后置处理器的postProcessBeforeInitialization方法

然后执行invokeInitMethods(beanName, wrappedBean, mbd);初始化方法

然后执行applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);方法，执行所有的后置处理器的postProcessAfterInitialization（）方法

**我们的BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；得到aspectJAdvisorsBuilder**

---

5) 把BeanPostProcessor注册到BeanFactory中

4 finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；

创建剩下的单实例bean， **刚才上面创建的是后置处理器bean**

```java
// Instantiate all remaining (non-lazy-init) singletons.
finishBeanFactoryInitialization(beanFactory);
```

初始化过程：

---

在finishBeanFactoryInitialization方法中，调用beanFactory.preInstantiateSingletons();创建方法

进入preInstantiateSingletons方法

 new ArrayList<String>(this.beanDefinitionNames)获取所有bean定义的bean名

遍历获取容器中所有的Bean，依次创建对象getBean(beanName);

getBean->doGetBean()->getSingleton()->

创建bean

先从缓存中获取当前bean（只要创建好的Bean都会被缓存起来），如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建

```java
@Override
public void preInstantiateSingletons() throws BeansException {
   if (this.logger.isDebugEnabled()) {
      this.logger.debug("Pre-instantiating singletons in " + this);
   }
   List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);

   // Trigger initialization of all non-lazy singleton beans...
   for (String beanName : beanNames) {
```

创建bean过程protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args)：

​	在createBean方法中

resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation

希望后置处理器在此能返回一个代理对象；如果能返回代理对象就返回

拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;

就执行**postProcessBeforeInstantiation**

- 【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】

- 【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】

  **AnnotationAwareAspectJAutoProxyCreator**实现的就是InstantiationAwareBeanPostProcessor后置处理器，所以，他会在任何bean创建之前，去尝试拦截

  在创建的时候它去调用

```java
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
```

如果不能就继续进入

doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)

这个才是真正的去创建一个bean实例，和之前说创建后置处理器bean一样

**所以， AnnotationAwareAspectJAutoProxyCreator，在所有bean创建之前会有一个拦截，因为实现了InstantiationAwareBeanPostProcessor，所以会调用postProcessBeforeInstantiation()返回一个代理的bean

---

5 **AopClass(我们的被切面的类)和LogAspects我们的切面类创建bean过程**

这两个bean是通过postProcessBeforeInstantiation来创建的

1 判断当前bean是否在advisedBeans中（保存了所有需要增强bean）

2 isInfrastructureClass(beanClass) 判断是否时切面类（AopClass不是）（@Aspect）

3 shouldSkip(beanClass, beanName)判断是否需要跳过

​	1）获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】



```java
@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		Object cacheKey = getCacheKey(beanClass, beanName);

		if (beanName == null || !this.targetSourcedBeans.contains(beanName)) {
			if (this.advisedBeans.containsKey(cacheKey)) {
				return null;
			}
			if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
				this.advisedBeans.put(cacheKey, Boolean.FALSE);
				return null;
			}
		}
		if (beanName != null) {.
				Object proxy = createProxy(beanClass, beanName, specificInterceptors, targetSource);
				this.proxyTypes.put(cacheKey, proxy.getClass());
				return proxy;
			}
		}

		return null;
	}
```

判断是否切面的方法

```java
public class AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator {
@Override
protected boolean isInfrastructureClass(Class<?> beanClass) {
   // Previously we setProxyTargetClass(true) in the constructor, but that has too
   // broad an impact. Instead we now override isInfrastructureClass to avoid proxying
   // aspects. I'm not entirely happy with that as there is no good reason not
   // to advise aspects, except that it causes advice invocation to go through a
   // proxy, and if the aspect implements e.g the Ordered interface it will be
   // proxied by that interface and fail at runtime as the advice method is not
   // defined on the interface. We could potentially relax the restriction about
   // not advising aspects in the future.
   return (super.isInfrastructureClass(beanClass) || this.aspectJAdvisorFactory.isAspect(beanClass));
}
```

创建完AopClass类之后，调用postProcessAfterInitialization方法

return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
 * 1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
      * 1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
      *2、获取到能在bean使用的增强器。
      *3、给增强器排序
 * 2）、保存当前bean在advisedBeans中；
 * 3）、如果当前bean需要增强，创建当前bean的代理对象；
      *1）、获取所有增强器（通知方法）
      * 2）、保存到proxyFactory
      * 3）、创建代理对象：Spring自动决定
           *JdkDynamicAopProxy(config);jdk动态代理；
           * ObjenesisCglibAopProxy(config);cglib的动态代理；
 * 		4）、给容器中返回当前组件使用cglib增强了的代理对象；
 * 		5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      if (!this.earlyProxyReferences.contains(cacheKey)) {
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```

目标方法执行:

容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
 * 1）、CglibAopProxy.intercept();拦截目标方法的执行
 * 2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
      * List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
      * 如何获取拦截器链
           * 1）、List<Object> interceptorList保存所有拦截器 5
                * 一个默认的ExposeInvocationInterceptor 和 4个增强器；
           * 2）、遍历所有的增强器，将其转为Interceptor；
                *registry.getInterceptors(advisor);
           * 3）、将增强器转为List<MethodInterceptor>；
                * 如果是MethodInterceptor，直接加入到集合中
                * 如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
                * 转换完成返回MethodInterceptor数组；

* 3）、如果没有拦截器链，直接执行目标方法;
     * 拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
 * 4）、如果有拦截器链，把需要执行的目标对象，目标方法，
      * 拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
      * 并调用 Object retVal =  mi.proceed();

*5）、拦截器链的触发过程（CglibMethodInvocation.proceed()方法过程 ）
     * 1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
          *2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
          *拦截器链的机制，保证通知方法与目标方法的执行顺序；
拦截器链时拍好续的，递归调用invoke方法，最后一个调用invoke，执行方法前的拦截器链，执行后返回，执行方法执行后的拦截器链，如果没抛出异常，返回后执行returning拦截器链

## 总结

 *1）、  @EnableAspectJAutoProxy 开启AOP功能
 * 2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
 * 3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
 * 4）、容器的创建流程：
    * 1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象

    * 2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean

       * 1）、创建业务逻辑组件和切面组件

       * 2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程

       * 3）、组件创建完之后，判断组件是否需要增强

         是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
 * 5）、执行目标方法：
    * 1）、代理对象执行目标方法
      *2）、CglibAopProxy.intercept()；
         * 	1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
         * 2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
            *3）、效果：
              * 正常执行：前置通知-》目标方法-》后置通知-》返回通知
              *  出现异常：前置通知-》目标方法-》后置通知-》异常通知

# 声明式事务

## 环境搭建

```java
@Configuration
@ComponentScan("com.xiao.tx")
//开始事务管理
@EnableTransactionManagement
public class TxConfig {

    //注入数据源
    @Bean
    public DataSource dataSource() throws Exception{
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUsername("root");

        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        String url = "jdbc:mysql://192.168.94.129:3306/mytest1" +
                "?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8";
        dataSource.setUrl(url);
        return dataSource;
    }
    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception{
        //传人dataSource(),Spring对@Configuration类会特殊处理；
        // 给容器中加组件的方法，多次调用都只是从容器中找组件
        return new JdbcTemplate(dataSource());
    }
    @Bean
    public PlatformTransactionManager transactionManager() throws Exception {
        return new DataSourceTransactionManager(dataSource());
    }
}
```

```java
@Service
public class UserService {
    @Autowired
    public UserDao userDao;
    @Transactional
    public void insert(){
        userDao.insert(UUID.randomUUID().toString());
        int i = 1/0;
    }
}
```

## 原理

@EnableTransactionManagement注解中import了TransactionManagementConfigurationSelector选择器

它导入了两个组件

 * AutoProxyRegistrar

 * ProxyTransactionManagementConfiguration

### AutoProxyRegistrar

 * 给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；

 * InfrastructureAdvisorAutoProxyCreator：

   利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
      implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
```

### ProxyTransactionManagementConfiguration

它是一个config

* 给容器中注册事务增强器
  * 事务增强器要用事务注解的信息，在AnnotationTransactionAttributeSource解析事务注解
  * 事务拦截器，
    * TransactionInterceptor：保存了事务属性信息，事务管理器；

```java
@Configuration
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {
    @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() 
        
        
        @Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionAttributeSource transactionAttributeSource() {
        //解析注解的信息
		return new AnnotationTransactionAttributeSource();
	}
    //事务拦截器
    @Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionInterceptor transactionInterceptor() {
		TransactionInterceptor interceptor = new TransactionInterceptor();
		interceptor.setTransactionAttributeSource(transactionAttributeSource());
		if (this.txManager != null) {
			interceptor.setTransactionManager(this.txManager);
		}
		return interceptor;
	}
```

* 从TransactionInterceptor可以看到，他是一个 MethodInterceptor（方法拦截器），在目标方法执行的时候，执行拦截器链；

```java
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable
```

* 从MethodInterceptor接口实现的invoke方法下的invokeWithinTransaction方法可以看到
  * 它先获取事务相关的属性
  * 再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmange(@Transactional(transactionManager = ""))，最终会从容器中按照类型获取一个PlatformTransactionManager

```java
	protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
			throws Throwable {

		// If the transaction attribute is null, the method is non-transactional.
		final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);
        //我们要执行的事务方法
        if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
// Standard transaction demarcation with getTransaction and commit/rollback calls.
            //开启事务
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
                //回滚事务
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}
            //提交事务
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}
```

# BeanFactoryPostProcessor

 * BeanPostProcessor：bean后置处理器，bean创建对象初始化前后进行拦截工作的
 * BeanFactoryPostProcessor：beanFactory的后置处理器；
    * 在BeanFactory标准初始化之后调用，来定制和修改BeanFactory的内容；
    * 所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建

```java
@Component //要想生效，则加入ioc容器中
public class MyBeanFactoryProcessor implements BeanFactoryPostProcessor {
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        //获取所有的bean信息
        String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
        System.out.println("MyBeanFactoryProcessor ....");
        for(String beanDefinitionName : beanDefinitionNames){
            System.out.println("MyBeanFactoryProcessor:"+beanDefinitionName);
        }
    }
}
```

```java
@ComponentScan("com.xiao.Ext")
@Configuration
public class ExtConfig {
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public TestInit testInit(){
        return new TestInit();
    }
}
```

## 原理

进入refresh() 方法，可以看到invokeBeanFactoryPostProcessors(beanFactory);方法，这个方法会找到所有BeanFactoryPostProcessor并执行,在invokeBeanFactoryPostProcessors方法中，执行invokeBeanFactoryPostProcessors方法

```java
class PostProcessorRegistrationDelegate {
   public static void invokeBeanFactoryPostProcessors(
   invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
```

```java
private static void invokeBeanFactoryPostProcessors(
      Collection<? extends BeanFactoryPostProcessor> postProcessors, ConfigurableListableBeanFactory beanFactory) {
    //获取BeanFactoryPostProcessor的名
    String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
    //获取所有未排序的BeanFactoryPostProcessor（源码上面还有其他排序的等，这里不贴出了）
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
   for (BeanFactoryPostProcessor postProcessor : postProcessors) {
      postProcessor.postProcessBeanFactory(beanFactory);
   }
}
```

# BeanDefinitionRegistryPostProcessor

* postProcessBeanDefinitionRegistry();
   - 在所有bean定义信息将要被加载，bean实例还未创建的；
   -  优先于BeanFactoryPostProcessor执行，并且先触发postProcessBeanDefinitionRegistry方法；
   -  利用BeanDefinitionRegistryPostProcessor给容器中再额外添加一些组件；

实例：

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("postProcessBeanDefinitionRegistry...");
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.rootBeanDefinition(Cat.class).getBeanDefinition();
        registry.registerBeanDefinition("hello", beanDefinition);
    }

    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("MyBeanDefinitionRegistryPostProcessor count:"+beanFactory.getBeanDefinitionCount());
    }
}
```

# Spring 事件

## 编程模型

- 观察者模式拓展
  - 消息发送者：java.util.Observable
  - 观察者：java.util.Observer
- 标准化接口(没有特殊规则，一般都是约束)
  - 事件对象：java.util.EventObject
  - 事件接听器：java.util.EventListener
- java示例

```java
public static void main(String[] args) {
    EventObservable eventObservable = new EventObservable();
    eventObservable.addObserver(new EventObserver());
    eventObservable.notifyObservers("发送某某消息");
}

static class EventObservable extends Observable {
    @Override
    public void notifyObservers(Object arg) {
        //需要将事件打开
        super.setChanged();
        super.notifyObservers(arg);
        super.clearChanged();
    }
}

static class EventObserver implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("收到事件： " + arg);
    }
}
```

## Spring标准事件

- org.springframework.context.ApplicationEvent 
- 拓展：org.springframework.context.event.ApplicationContextEvent

## Spring监听器

### 基于接口

- 基于EventListener扩展
- 扩展接口：org.springframework.context.ApplicationListener
- 处理单一类型事件

- 示例

```java
GenericApplicationContext applicationContext = new GenericApplicationContext();
applicationContext.addApplicationListener(new ApplicationListener<ApplicationEvent>() {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("收到事件:"+event);
    }
});
applicationContext.refresh();
applicationContext.start();
applicationContext.close();
```

- 打印日志

```
---
org.springframework.context.event.ContextRefreshedEvent
----
org.springframework.context.event.ContextStartedEvent
-----
org.springframework.context.event.ContextClosedEvent
```

### 基于注解



- 示例

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(AnnotationListenerDemo.class);

    applicationContext.refresh();
    applicationContext.close();
}

@EventListener
public void commonEvent(ApplicationEvent event) {
    System.out.println("事件:"+event);
}
```

- 专属事件

```java
@EventListener
public void refreshEvent(ContextRefreshedEvent event) {
    System.out.println("refresh事件:"+event);
}
```

## 事件发布器

- org.springframework.context.ApplicationEventPublisher	
  - 依赖注入
  - 示例

```java
public class AnnotationListenerDemo implements ApplicationEventPublisherAware {
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        applicationEventPublisher.publishEvent(new ApplicationEvent("发布事件") {
        });
    }
}
```

- org.springframework.context.event.ApplicationEventMulticaster
  - 依赖注入
  - 依赖查找

### 注入ApplicationEventPublisher	

- 通过ApplicationEventPublisherAware接口（ApplicationContext是ApplicationEventPublisher的子类，也可以通过它来发布，但是注入顺序可以参考aware执行顺序）
- 通过@Autowired注入ApplicationEventPublisher

## 原理

- 核心类：org.springframework.context.event.SimpleApplicationEventMulticaster
- 添加事件：AbstractApplicationEventMulticaster#addApplicationListener

### 事件发布流程

-  进入refresh()方法，执行finishRefresh();方法，再执行publishEvent(new ContextRefreshedEvent(this));发布事件
-  进入getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);方法（获取事件的多播器（派发器）：getApplicationEventMulticaster()）
-  进入multicastEvent()方法派发事件

```java
@Override
public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
   ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
   for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
      Executor executor = getTaskExecutor();
      if (executor != null) {
         executor.execute(new Runnable() {
            @Override
            public void run() {
               invokeListener(listener, event);
            }
         });
      }
      else {
         invokeListener(listener, event);
      }
   }
}
```

### 事件多播器

在refresh（）方法中，在其他bean创建之前，执行initApplicationEventMulticaster();方法，来初始化事件多播器

## 注解的方式建立监听

```java
@Component
public class MyListener {
    @EventListener(classes = ApplicationEvent.class)
    public void listener(ApplicationEvent event){
        System.out.println("收到注解的事件监听:"+event);
    }
}
```

## 事件异常处理

- 当事件出现异常是，处理信息

- 在SimpleApplicationEventMulticaster中有个ErrorHandler属性
- 需要调用SimpleApplicationEventMulticaster#setErrorHandler方法才会生效

# Spring 生命周期

## 上下文启动准备

- AbstractApplicationContext#prepareRefresh

```java
//记录启动时间，状态标识
this.startupDate = System.currentTimeMillis();
this.closed.set(false);
this.active.set(true);
//拓展属性，默认是没有，可以由子类拓展一些属性，如：web的一些属性
initPropertySources();
```

## Beanfactory创建阶段

- AbstractApplicationContext#obtainFreshBeanFactory
- 调用AbstractRefreshableApplicationContext#refreshBeanFactory

```java
protected final void refreshBeanFactory() throws BeansException {
    //如果有Beanfactory,则进行销毁
   if (hasBeanFactory()) {
      destroyBeans();
      closeBeanFactory();
   }
   try {
      //创建beanfactory
      DefaultListableBeanFactory beanFactory = createBeanFactory();
      beanFactory.setSerializationId(getId());
      //是否允许beandefinition重复定义和是否允许循环依赖的属性设置
      customizeBeanFactory(beanFactory);
      //从注解，或者xml中加载beandefinition的信息
      loadBeanDefinitions(beanFactory);
      synchronized (this.beanFactoryMonitor) {
         this.beanFactory = beanFactory;
      }
   }
}
```

## 准备阶段

- AbstractApplicationContext#prepareBeanFactory
- 添加aware（通过addBeanPostProcessor）
- 忽略aware的注入

```java
//添加aware
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
```

- 依赖注入BeanFactory等

## 后置处理阶段

- AbstractApplicationContext#postProcessBeanFactory 
  - 依赖子类来实现他的拓展
  - 子类可以通过这个方式来添加aware接口如：GenericWebApplicationContext#postProcessBeanFactory方法
- AbstractApplicationContext#invokeBeanFactoryPostProcessors
  - 执行BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry
  - 然后再执行BeanFactoryPostProcessor#postProcessBeanFactory

## 注册BeanpostProcessor

- AbstractApplicationContext#registerBeanPostProcessors

- 由源码可以看出，如果BeanPostProcessor标注了PriorityOrdered，那么他会提前依赖查找，但是这个初始化可能bean准备不是那么的充分

```java
if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
   BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   priorityOrderedPostProcessors.add(pp);
   if (pp instanceof MergedBeanDefinitionPostProcessor) {
      internalPostProcessors.add(pp);
   }
}
```

## 初始化MessageSource

- 内建Bean(国际化文案)
- AbstractApplicationContext#initMessageSource

```java
//判断MessageSource是否已构建好，springboot中，会在外部创建这个bean
if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
   this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
   }
}
```

## Spring 广播事件初始化

- 内建bean
- AbstractApplicationContext#initApplicationEventMulticaster
- 最终变为一个单例的bean

## 上下文刷新阶段

- AbstractApplicationContext#onRefresh
- 提供子类来扩展
- 应用于web项目

## 事件监听器注册

- AbstractApplicationContext#registerListeners

## 初始化完成阶段

- AbstractApplicationContext#finishBeanFactoryInitialization
- 当ConversionService存在时，关联ConversionService
- 添加StringValueResolver
  - 处理一系列字符串的逻辑比如：占位符解释、SpEL计算等等
- 初始化单例Beans



## 应用上下文关闭阶段

- AbstractApplicationContext#close 
- 状态表示关闭:active(fales), closed(true)
- 发布Spring应用上下文关闭时间
- 注销Shutdown Hook（优雅关闭线程能够触发close事件）

# SmartInitializingSingleton

在初始化容器，创建完所有单实例非懒加载bean之后，会执行实现了SmartInitializingSingleton接口的bean的afterSingletonsInstantiated方法，具体可见refresh()方法里的finishBeanFactoryInitialization(beanFactory);方法

```
public interface SmartInitializingSingleton {
   void afterSingletonsInstantiated();
}
```

# Spring容器的refresh()

1 prepareRefresh()刷新前的预处理;

- initPropertySources()初始化一些属性设置;子类自定义个性化的属性设置方法；

- getEnvironment().validateRequiredProperties();检验属性的合法等

- earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();保存容器中的一些早期的事件

2 obtainFreshBeanFactory();获取BeanFactory；

- refreshBeanFactory();刷新【创建】BeanFactory；

  - 创建了一个this.beanFactory = new DefaultListableBeanFactory();

  - 设置一个序列化id；

- getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
- 将创建的BeanFactory【DefaultListableBeanFactory】返回；

3 prepareBeanFactory(beanFactory);BeanFactory的预准备工作（BeanFactory进行一些设置）；

- 设置BeanFactory的类加载器、支持表达式解析器...
- 添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
- 设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
- 注册可以解析的自动装配；我们能直接在任何组件中自动注入：BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
- 添加BeanPostProcessor【ApplicationListenerDetector】
- 添加编译时的AspectJ；
- 给BeanFactory中注册一些能用的组件(我们要使用直接autowied就可以，因为在这里注入进来了)；environment【ConfigurableEnvironment】、systemProperties【Map<String, Object>】、systemEnvironment【Map<String, Object>】

4 invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessor的方法；

- BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的
- 它有两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor
- 先执行BeanDefinitionRegistryPostProcessor的方法；
- 再执行BeanFactoryPostProcessor的方法

5 registerBeanPostProcessors(beanFactory);注册（仅仅只是将其注册进入beanfactory中）BeanPostProcessor（Bean的后置处理器）：拦截bean的创建过程

- 获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
   - BeanPostProcessor有这些接口：每个接口执行时间不同
       - BeanPostProcessor、DestructionAwareBeanPostProcessor、InstantiationAwareBeanPostProcessor、SmartInstantiationAwareBeanPostProcessor、MergedBeanDefinitionPostProcessor
- 先注册PriorityOrdered优先级接口的BeanPostProcessor，把每一个BeanPostProcessor；添加到BeanFactory中
- 最后注册没有实现任何优先级接口的
- 注册MergedBeanDefinitionPostProcessor
- 最后注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，这个家伙是用来检测那个时监听器的，如果是，添加到容器中

6 initMessageSource();初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；

- 把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；
- MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);

7 initApplicationEventMulticaster();初始化事件派发器

- 将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入

8 registerListeners();给容器中将所有项目里面的ApplicationListener注册进来

- 将每个监听器添加到事件派发器中
- 派发之前步骤产生的事件

9 finishBeanFactoryInitialization(beanFactory);初始化所有剩下的单实例bean

- beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean
	-  获取容器中的所有Bean，依次进行初始化和创建对象
	-  获取Bean的定义信息；RootBeanDefinition
	- 判断 Bean不是抽象的，是单实例的，是懒加载
		- 判断是否是FactoryBean；是否是实现FactoryBean接口的Bean（工厂bean的话直接调用getobject来获取bean，之前笔记有）
		- getBean(beanName)；方法，其实就是 ioc.getBean();，之前我们用context.getBean()就是这个
		- 先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来），从private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);获取的
		- 缓存中获取不到，开始Bean的创建对象流程
		- 获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；比如之前我们有在在配置文件中配置dependsOn
		<bean id="testBean" class="com.xiao.entry.TestBean" depends-on="book" >
		- String[] dependsOn = mbd.getDependsOn();
		- 启动单实例Bean的创建流程

```java
// Create bean instance.
if (mbd.isSingleton()) {
   sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
      @Override
      public Object getObject() throws BeansException {
         try {
            return createBean(beanName, mbd, args);
         }
         catch (BeansException ex) {
            destroySingleton(beanName);
            throw ex;
         }
      }
   });
   bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

进入createBean(beanName, mbd, args)方法

进入resolveBeforeInstantiation方法，尝试获取BeanPostProcessors（InstantiationAwareBeanPostProcessor类型）的后置方法，方法代理对象

先触发：postProcessBeforeInstantiation()；

如果有返回值：触发postProcessAfterInitialization()；

```java
try {
			//试获取BeanPostProcessors的后置方法，方法代理对象
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}
		//创建单实例bean
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		if (logger.isDebugEnabled()) {
			logger.debug("Finished creating instance of bean '" + beanName + "'");
		}
```



---

进入Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean方法创建bean

【创建Bean实例】；createBeanInstance(beanName, mbd, args)

applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName)：调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition(mbd, beanType, beanName);

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
      throws BeanCreationException {
   if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }
   synchronized (mbd.postProcessingLock) {
	if (!mbd.postProcessed) {
		try {
            //创建了bean，未初始化之前
            //MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition
		    applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
		}
	Object exposedObject = bean;
	try {
        //进行属性赋值
		populateBean(beanName, mbd, instanceWrapper);
```

进入【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);方法

赋值之前：

拿到InstantiationAwareBeanPostProcessor后置处理器；执行postProcessAfterInstantiation()；

拿到InstantiationAwareBeanPostProcessor后置处理器；postProcessPropertyValues()；**我们可以利用这个来额外的设置一些属性**

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
	for (BeanPostProcessor bp : getBeanPostProcessors()) {
	if (bp instanceof InstantiationAwareBeanPostProcessor) {
		InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                   //执行后置处理器方法
		if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
           
```

赋值前工作完成

进入 applyPropertyValues(beanName, mbd, bw, pvs);方法，进行一些属性的赋值

**回到doCreateBean方法**

进行bean初始化工作

```java
try {
   populateBean(beanName, mbd, instanceWrapper);
   if (exposedObject != null) {
      //进行bean的初始化
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }
}
```

进入initializeBean方法

【执行Aware接口方法】invokeAwareMethods(beanName, bean);执行xxxAware接口的方法

​		BeanNameAware\BeanClassLoaderAware\BeanFactoryAware

在初始化之前，执行后置处理器beanProcessor.postProcessBeforeInitialization(result, beanName);



【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);

执行初始化之后方法

applyBeanPostProcessorsAfterInitialization

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    else {
			invokeAwareMethods(beanName, bean);
	}
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    try {
		invokeInitMethods(beanName, wrappedBean, mbd);
	}
    if (mbd == null || !mbd.isSynthetic()) {
	wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
	}
```

**回到doCreateBean方法**, 执行注册Bean的销毁方法



ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。



所有Bean都利用getBean创建完成以后；
检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；

# spring初始化总结

1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
		1）、xml注册bean；<bean>
		2）、注解注册Bean；@Service、@Component、@Bean、xxx
	2）、Spring容器会合适的时机创建这些Bean
		1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
		2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
	3）、后置处理器；BeanPostProcessor
		1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
			AutowiredAnnotationBeanPostProcessor:处理自动注入
			AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
			xxx....
			增强的功能注解：
			AsyncAnnotationBeanPostProcessor
			....
	4）、事件驱动模型；
		ApplicationListener；事件监听；
		ApplicationEventMulticaster；事件派发：

# Spring Environment

- 统一的Spring配置属性管理

## 使用场景

- PropertyPlaceholderConfigurer
  - 3.1时代使用

1. 建立一个/META-INF/person.properties， 属性为name=xxx
2. 使用PropertyPlaceholderConfigurer加载

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register(PropertyPlaceholderConfigurerDemo.class);
    context.refresh();
    Person bean = context.getBean(Person.class);
    System.out.println(bean);
    context.close();
}

@Bean
public PropertyPlaceholderConfigurer propertyPlaceholderConfigurer(@Value("classpath:/META-INF/person.properties") Resource resource) {
    PropertyPlaceholderConfigurer configurer = new PropertyPlaceholderConfigurer();
    configurer.setFileEncoding("UTF-8");
    configurer.setLocation(resource);
    return configurer;
}

@ToString
@Setter
static class Person {
    @Value("${name}")
    private String name;
}

@Bean
public Person person() {
    return new Person();
}
```

## Evironment依赖注入

- 通过EnvironmentAware接口回调
- 通过@Autowired
- ApplicationContext#getEnvironment获取



## Environment依赖查找

```java
@Autowired
private Environment environment;

public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.register(LookupEnvironmentDemo.class);
    applicationContext.refresh();
    //依赖查找
    Environment environmentTmp = applicationContext.getBean(ConfigurableApplicationContext.ENVIRONMENT_BEAN_NAME, Environment.class);
    System.out.println(environmentTmp == applicationContext.getBean(LookupEnvironmentDemo.class).environment);
    applicationContext.close();
}
```

## @PropertySource 原理

- 入口方法：ConfigurationClassParser#doProcessConfigurationClass

# Enable编程模型

## 简单模式

- ps:Configuration如果在componentscan里面，一样会生效，所以不建议这样写
- 定义一个config

```java
@Configuration
public class HelloWordConfig {

    @Bean
    public String helloWord() {
        return "helloWord";
    }
}
```

- 编写enbale注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(HelloWordConfig.class)
public @interface EnableHelloWord {

}
```

- 编写启动类，启动enable相关bean

```java
@EnableHelloWord
public class HelloWordBootStrap {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.register(HelloWordBootStrap.class);
        applicationContext.refresh();
        String bean = applicationContext.getBean("helloWord", String.class);
        System.out.println(bean);
        applicationContext.close();
    }
}
```

## ImportSelector模式

- 模拟启动服务

- 建立一个接口，下面实现两个实现类，分别是ftp服务，http服务

```java
public interface Server {
    /**
     * 启动
     */
    void start();
    /**
     * 停止
     */
    void stop();
    enum Type {
         FTP,
        HTTP,
        ;
    }
}
```

- 定义一个selector,如果启动ftp，则返回ftp的实现类

```java
public class ServerSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        Map<String, Object> annotationAttributes = importingClassMetadata.getAnnotationAttributes(EnableServer.class.getName());
        Server.Type type = (Server.Type) annotationAttributes.get("type");
        if(Server.Type.FTP.equals(type)) {
            return new String[] {FtpServerImpl.class.getName()};
        } else {
            return new String[] {HttpServerImpl.class.getName()};
        }
    }
}
```

- 定义一个注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
@Import(ServerSelector.class)
public @interface EnableServer {

    Server.Type type();
}
```

## ImportBeanDefinition

- 使用beandefinition的方式

```java
public class ServerBeanDefinition implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(HttpServerImpl.class.getName());
        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinition, registry);
    }
}
```

- 引入

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(HelloWordConfig.class)
```
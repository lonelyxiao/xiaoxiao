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
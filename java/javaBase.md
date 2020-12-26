# 注解

Java 语言中的类、方法、变量、参数和包等都可以被标注

## 文档类型注解(编写文档)

```java
/**
 *  注解javadoc演示
 *  @author xiao
 *  @version 1.o
 *  @since 1.5
 */
public class AnnoDemo1 {
    /**
     * 计算两个数的和
     * @param a 加数
     * @param b 加数
     * @return 两个数的和
     */
    public int add(int a, int b) {
        return a+b;
    }
}
```

## 预定义注解

**@Override**

检测方法是否继承父类

**@Deprecated**

标注内容已过时

**@SuppressWarnings**

压制警告，一般传递参数 all

## 自定义注解

### 格式

元注解

public @interface 注解名称 ｛｝

```java
public @interface MyAnno {
}
```

注解本质是一个继承了**Annotation接口**的接口

### 属性

接口可以定义的内容（成员方法）

要求

返回类型必须是：**基本数据类型，String， 枚举，注解，以上数据类型的数组**

定义了属性以后，使用属性时必须赋值

```java
public @interface MyAnno {
    int age();
}
```

```java
@MyAnno(age = 11)
```

如果不想赋值

```java
int age() default 1;
```

如果只有一个属性，则将属性名定义value，则属性值默认就value

```java
public @interface MyAnno {
    int value() default 1;
}
```

```java
@MyAnno(11)
```

### 元注解

用于描述注解的注解

RetentionPolicy一般都是runtime阶段

- @Target:注解的作用目标　　　　　　　　

　　　　@Target(ElementType.TYPE)   //接口、类、枚举、注解

　　　　**@Target(ElementType.FIELD) //字段、枚举的常量**

　　　　**@Target(ElementType.METHOD) //方法**

　　　　**@Target(ElementType.PARAMETER) //方法参数**

　　　　**@Target(ElementType.CONSTRUCTOR)  //构造函数**

　　　　**@Target(ElementType.LOCAL_VARIABLE)//局部变量**

　　　　**@Target(ElementType.ANNOTATION_TYPE)//注解**

　　　　**@Target(ElementType.PACKAGE) ///包**  



```java
@Target(ElementType.METHOD) //注解作用范围
@Retention(RetentionPolicy.SOURCE) //注解被保留的阶段
@Documented//注解是否被抽取到文档api
@Inherited //是否被继承
```

### 程序中解析注解

注解其实就是用来取动态的配置的值得

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnno {
    int value() default 1;

    String className();

    String method();
}
```

获取注入的值

```java
@MyAnno(method = "show", className = "com.xiao.stu_es.annotation.MyAnno")
public class Test {
    public static void main(String[] args) {
        Class<Test> testClass = Test.class;
        MyAnno annotation = testClass.getAnnotation(MyAnno.class);
        //获取注解注入的值
        String className = annotation.className();
        String method = annotation.method();
        System.out.println(className);
        System.out.println(method);
    }
}
```


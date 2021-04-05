# 泛型

## 元组

- 将一组对象直接打包存储于单一对象中。可以从该对象读取其中的元素，但不允许向其中存储新对象

- 下面是一个可以存储两个对象的元组
  - 为什么a1 a2 是private类型
  - 元组的使用程序可以读取 `a1` 和 `a2` 然后对它们执行任何操作，但无法对 `a1` 和 `a2` 重新赋值。例子中的 `final` 可以实现同样的效果，并且更为简洁明了。

```java
public class Tuple2<A, B> {
    public final A a1;
    public final B a2;
    public Tuple2(A a, B b) { a1 = a; a2 = b; }
}
```

## 集合工具

- 前三个方法通过将第一个参数的引用复制到新的 **HashSet** 对象中来复制第一个参数，因此不会直接修改参数集合。因此，返回值是一个新的 **Set** 对象。

```java
public class Sets {
    // 并集
    public static <T> Set<T> union(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet<>(a);
        result.addAll(b);
        return result;
    }
	//交集
    public static <T> Set<T> intersection(Set<T> a, Set<T> b) {
        Set<T> result = new HashSet<>(a);
        result.retainAll(b);
        return result;
    }
	// 从 superset 中减去 subset 的元素
    public static <T> Set<T> difference(Set<T> superset, Set<T> subset) {
        Set<T> result = new HashSet<>(superset);
        result.removeAll(subset);
        return result;
    }

    public static <T> Set<T> complement(Set<T> a, Set<T> b) {
        return difference(union(a, b), intersection(a, b));
    }
}
```

## 泛型擦除

- Java的泛型是伪泛型，这是因为Java在编译期间，所有的泛型信息都会被擦掉
- 

```java
public static void main(String[] args) {
    Class c1 = new ArrayList<String>().getClass();
    Class c2 = new ArrayList<Integer>().getClass();
    System.out.println(c1 == c2);
}
```

- 输出结果为true
- 无论两个T是啥类型，他们的class都是相同的

- 如下，编译器是无法通过编译的，因为 obj.f(); obj是不知道什么类型的，只有改成Manipulator2<T extends HasF>，才能通过编译，因为泛型会擦除到边界（HasF）

```java
class Manipulator<T> {
    private T obj;
    
    Manipulator(T x) {
        obj = x;
    }
    public void manipulate() {
        obj.f();
    }
}
public class Manipulation {
    public static void main(String[] args) {
        HasF hf = new HasF();
        Manipulator<HasF> manipulator = new Manipulator<>(hf);
        manipulator.manipulate();
    }
}
```



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

- 使用元组时，你只需要定义一个长度适合的元组，将其作为返回值即可

```java
static Tuple2<String, Integer> f() {
    return new Tuple2<>("hi", 47);
}
public static void main(String[] args) {
    Tuple2<String, Integer> ttsi = f();
    System.out.println(k());
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

# JAVA9

## 新特性

- jdk9目录不包含jre
- 模块化系统

## JSHELL

- REPL工具：jShell命令
  - 以交互式的方式，对语句和表达式进行求值
  - 如同scala之类一样
- 帮助文档：/help

```shell
##进入jshell
λ ./jshell
## 执行java代码
jshell> System.out.println("hello word");
hello word

## 导入java包，导入后可以调用对应包下的方法
jshell> import java.util.*
## 查看已导入的包
jshell> /import
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*
|    import java.util.*

## 查看已输入的语句
jshell> /list

   1 : System.out.println("hello word");
   2 : import java.util.*;
```

1. 导入java文件

```java
void printHello() {
    System.out.println("hello word! java9");
}
printHello();
```

2. 打开编写的文件

```shell
jshell> /open D:\softinstall\jdk-9\bin\HelloWord.java
hello word! java9
```

- 多版本兼容jar包
  - 在对于版本下使用对应版本的class

## 模块化

- 好处：安全、加载更快一点

1. 建立一个模块exprot-module，这个模块是用来导出的，在根路径下简历module-info.java文件

```java
module exprot.module {
    exports com.xiao.exprot ;
}
```

2. 建立实体类(用于在另一个模块引入)

```java
package com.xiao.exprot;

public class Person {
    private String name;
    private Integer age;
}
```

3. 建立引入模块，建立module-info.java文件

```java
module improt.module{
    requires exprot.module;
}
```

4. test使用

```java
public class Test {
    public static void main(String[] args) {
        Person person = new Person();
    }
}
```

## 接口私有方法

- http://openjdk.java.net/jeps/213
- 为啥会出现：jdk8出现了接口方法可以写方法体，方法可以调用，则出现了private类型

```java
interface MyInterface {
    // jdk7
    void method1();

    //jdk8: 可以定义static方法和default方法
    static void method2() {
        System.out.println("method 2");
    }

    default void method3() {
        System.out.println("method3");
    }
    //jdk9: 可以定义private方法
    private void method4() {
        System.out.println("method4");
    }
}
```

## 钻石操作符提升

```java
public void DiamondMethod() {
    new HashMap<>() {
        //可以在子类的匿名方法中编写代码
        @Override
        public Object get(Object key) {
            //重写父类的方法等操作
            return super.get(key);
        }
    };
}
```

## String 类型由byte数组存储，由**coder**存储字符编码

- 大部分的string存储的是拉丁文，这样char一样占用了两个字节，这样浪费了空间
- 使用byte就不会这样问题
- 如果不是拉丁文，用utf-16存储

## 创建只读集合

- java8和java9对比

```java
// java8
List<String> list1 = new ArrayList<>();
list1.add("a");
List<String> list2 = Collections.unmodifiableList(list1);
//java9
List<String> list3 = List.of("a", "b");
```

## Strem提升

- takeWhile操作

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
//如果满足条件则通过，当第一次不满足时，则终止循环
list.stream().takeWhile(num -> num > 3).forEach(System.out::println);
//返回剩余的
list.stream().dropWhile(num -> num>3).forEach(System.out::println);
//iterate多了个重载方法来判断是否停止
Stream.iterate(0, x -> x < 10, x -> x+1).forEach(System.out::println);
//Optional多了一个Stream方法，返回一个集合，可以调用flatmap变为集合操作
Optional.ofNullable(Arrays.asList(1,2,3,4)).stream().forEach(System.out::println);
```

## HTTP/2 Client

- 对应110 http://openjdk.java.net/jeps/110
- 使用姿势

1. 引入模块

```java
module stu.java9 {
    requires jdk.incubator.httpclient;
}
```

2. 使用

```java
public static void main(String[] args) throws IOException, InterruptedException {
    HttpClient httpClient = HttpClient.newHttpClient();
    HttpRequest httpRequest = HttpRequest.newBuilder(URI.create("https://www.baidu.com")).GET().build();

    HttpResponse<String> httpResponse = httpClient.send(httpRequest, HttpResponse.BodyHandler.asString());

    System.out.println("status:"+ httpResponse.statusCode());
    System.out.println("Http version: "+ httpResponse.version());
    System.out.println("Http body: \n" + httpResponse.body());
}
```

## 默认使用G1垃圾回收

# JAVA11

## 局部变量的类型推断

- var其实就是从右边推断类型，并不是弱类型
- 它的作用可以看做：定义当一个很长的类名时，我们可以用var来代替

```shell
jshell> var a = "hello";
a ==> "hello"
jshell> System.out.println(a.getClass());
class java.lang.String
```


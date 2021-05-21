# 泛型

## 元组

- 将一组对象直接打包存储于单一对象中。可以从该对象读取其中的元素，但不允许向其中存储新对象

### get方法的另一种优化

- 下面是一个可以存储两个对象的元组
  - 为什么a1 a2 不是private类型（安装java规范，应该是定义private类型，然后定义getset方法来操作）
  - 元组的使用程序可以读取 `a1` 和 `a2` 然后对它们执行任何操作，但无法对 `a1` 和 `a2` 重新赋值。例子中的 `final` 可以实现同样的效果，并且更为简洁明了。
  - 这里不可以再对元祖赋值了，只有初始化的时候可以赋值，所以可以这样优化代码

```java
public class Tuple2<A, B> {
    public final A a1;
    public final B a2;
    public Tuple2(A a, B b) { a1 = a; a2 = b; }
}
```



## 泛型擦除

- Java的泛型是使用擦除实现的 ，这是因为Java在编译期间，所有的泛型信息都会被擦掉
- 因此， List<String> 和 List<Integer> 在运行时实际上是相同的类型。它们都被擦除成原生类型 List 
- 无论两个T无论是啥类型，他们的class都是相同的

```java
//结果输出为：true
public static void main(String[] args) {
    Class c1 = new ArrayList<String>().getClass();
    Class c2 = new ArrayList<Integer>().getClass();
    System.out.println(c1 == c2);
}
```

### 泛型边界

- 如下，编译器是无法通过编译的，因为 obj.f(); obj是不知道什么类型的，只有改成Manipulator2<T extends HasF>，才能通过编译，因为泛型会擦除到边界（HasF），T 擦除到了 HasF 

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

- 如下，则显示false
  - 因为编译的时候，泛型擦除到第一个边界 Integer/String

```java
static class Border1<T extends Integer> {
}
static class Border2<T extends String> {
}
public static void main(String[] args) {
    Border1<Integer> border1 = new Border1<>();
    Border2<String> border2 = new Border2<>();
    System.out.println(border1.getClass().equals(border2.getClass()));
}
```

# 集合

## Set工具Api

- 定义一个set工具

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



# 文件

## Path

- Paths#get(java.lang.String, java.lang.String...)

- 绝对路径转为相对路径

```java
//toAbsolutePath：获取项目上两层的目录
//normalize：将相对路径转为绝对路径
System.out.println(Paths.get("..", "..").toAbsolutePath().normalize());
```

- 移除path的根路径

```java
//D:\git\gitee
Path path = Paths.get("..", "..").toAbsolutePath().normalize();
//D:\git\gitee\learning\stu-java-base\TestPath.java
Path path1 = Paths.get("TestPath.java").toAbsolutePath();
//learning\stu-java-base\TestPath.java
System.out.println(path.relativize(path1));
```

- 拼接地址, 将原有的地址拼接other字符串，再返回一个新的地址

```java
Path resolve(String other);
```

- 判断起始路径

```java
System.out.println(testTmp.startsWith("D:\\"));
```

## 文件系统

- 获取文件系统

```java
FileSystem fsys = FileSystems.getDefault();
```

- 获取操作系统的盘符：fsys.getFileStores()
- 获取根目录集合：fsys.getRootDirectories()
- 获取系统文件目录分隔符：fsys.getSeparator()

### 路径监听

```java
//监听这个文件夹下的文件
Path resolve = testTmp.resolve("test");
WatchService watcher = FileSystems.getDefault().newWatchService();
resolve.register(watcher, ENTRY_DELETE);
//watcher.take() 将等待并阻塞在这里。当目标事件发生时，会返回一个包含 WatchEvent 的 Watchkey 对象
WatchKey key = watcher.take();
for(WatchEvent evt : key.pollEvents()) {
    System.out.println("evt.context(): " + evt.context() +
            "\nevt.count(): " + evt.count() +
            "\nevt.kind(): " + evt.kind());
    System.exit(0);
}
```

## 文件读写



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

# JAVA8

## 环境

新项目：project  sdk(1.8)   level(8)->modules level（8）->java compile version(1.8)



## 特性

### Lambda表达式

其本质是接口（只有一个方法的接口）的实现

### 举例

格式：

- -> :lambda操作符
- (o1,o2)：lambda形参列表 其实就是抽象方法的形参
- Integer.compare(o1,o2) ： lambda体： 就是重写的抽象方法的方法体

```java
@Test
public void test1(){
    Comparator<Integer> comparator = new Comparator<Integer>() {
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1,o2);
        }
    };
    System.out.println(comparator.compare(1,2));

    Comparator<Integer> comparator1 = (o1,o2) -> Integer.compare(o1,o2);
    System.out.println(comparator1.compare(1,2));
}
```

### Lambda 表达式语法

第一种方式，没有参数

```java
Runnable runnable = () -> {
    System.out.println("第一种表达方式，没有参数");
};
runnable.run();
```

第二种表达式，有形参没有返回值

```java
Consumer<String> consumer = new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
};
consumer.accept("人生真是无奈啊");
Consumer<String> consumer1 = (String s)-> System.out.println(s);
consumer1.accept("还是要调整心态");

//优化：类型推断，泛型里传了String，形参推荐是String
Consumer<String> consumer2 = (s)-> System.out.println(s);
```

第三种：如果只有一个参数，可以省略括号

```java
Consumer<String> consumer2 = s-> System.out.println(s);
```

第四种情况：如果有多行语句，return和{}不能省略

### 函数式接口

如果一个接口只声明了一个接口方法，那么它就是函数式接口

### java四大函数式接口

- 消费型接口 Consumer<T>     void accept(T t)
  - 消费型接口，只有入参没有返回值
- 供给型接口 Supplier<T>     T get()
- 函数型接口 Function<T,R>   R apply(T t)
- 断定型接口 Predicate<T>    boolean test(T t)
  - 如果存在某个值，返回true



Consumer：消费型接口，只有入参没有返回值

```java
@Test
public void Test2() {
    Double price = 9.99;
    consumer(price, new Consumer<Double>() {
        @Override
        public void accept(Double aDouble) {
            System.out.println("我消费了："+aDouble);
        }
    });
    //相当于我传值和代码进去，做什么操作你给我执行
    consumer(price, aDouble-> System.out.println("我又消费了："+aDouble));
}

public void consumer(Double price, Consumer<Double> consumer){
    consumer.accept(price);
}
```

Predicate:如果存在某个值，返回true

```java
@Test
public void Test3(){
    Predicate<String> predicate = new Predicate<String>() {
        @Override
        public boolean test(String s) {
            return s.contains("a");
        }
    };
    Predicate<String> predicate1 = s -> s.contains("a");
    List list = predicate(Arrays.asList("a", "bb", "ab"), predicate1);
    System.out.println(Arrays.asList(list).toString());
}

public List<String> predicate(List<String> list, Predicate<String> predicate){
    List<String> list1 = new ArrayList();
    for(String str : list){
        if(predicate.test(str)){
            list1.add(str);
        }
    }
    return list1;
}
```

## 方法引用

方法引用可以看做是Lambda表达式深层次的表达

要求：实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致！

使用情况：

- 对象::实例方法名

```java
Consumer<String> consumer = x -> System.out.println(x);
//方法引用使用方式
PrintStream out = System.out;
//out为对象，调用println方法
Consumer<String> consumer1 = out::println;
consumer1.accept("hahah");
```

- 类::静态方法名

```java
//java使用方式
Comparator<Integer> comparator = (x,y)-> Integer.compare(x,y);
//方法引用
Comparator<Integer> comparator1 = Integer::compare;
int compare = comparator1.compare(1, 2);
```

- 类::实例方法名(形参列表可以不一致)

注意：当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时：ClassName::methodName

```java
BiPredicate<String, String > biPredicate = (y, x)-> y.equals(x);
BiPredicate<String, String > biPredicate1 = String::equals;
biPredicate1.test("1","1");
```

```java
//传入user对象，返回字符串
Function<User, String> function = u->u.getUsername();
String s = function.apply(new User("萧", "2222"));
Function<User, String> function1 = User::getUsername;
System.out.println(function1.apply(new User("萧", "2222")));
```

## 构造器引用

构造器和方法的引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一直。抽象方法的返回值类型即为构造器所属的类的类型

- 无参构造

```java
//T get();无参，new出来的为空参构造器
Supplier<User> supplier = ()->new User();
Supplier<User> supplier1 = User::new;
```

- 有参构造器

```java
//R apply(T t); 传入String 类型，构造器参数也是一个String
Function<String, User> function = x->new User(x);
Function<String, User> function1 = User::new;
function1.apply("耨人");
```

- 数组引用

```java
Function<Integer, String[]> function2 = length->new String[length];
Function<Integer, String[]> function3 = String[]::new;
```

## Stream

Stream 关注的是计算

Stream 自己不会存储元素。

Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。

Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行

### Stream 执行流程

- Stream的实例化
- 一系列的中间操作（过滤、映射、...)
- 终止操作

说明：

- 4.1 一个中间操作链，对数据源的数据进行处理
- 4.2 一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用

### 创建Stream的方式

- 从集合中获取

```java
List<User> users = UserData.getUsers();
//获取顺序流
Stream<User> stream = users.stream();
//获取并发流
Stream<User> userStream = users.parallelStream();
```

- 通过数组

```java
int [] arr = {1,2,3,4};
IntStream stream1 = Arrays.stream(arr);
User[] users1 = {new User("1", "2")};
Stream<User> stream2 = Arrays.stream(users1);
```

- 通过steam.of

```java
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4);
```

## Stream穿件无限流

- static <T> Stream<T> generate(Supplier<T> s)
- 可以利用泛型的方法来创建无限流(示例来源：on java8)

```java
public class BasicSupplier<T> implements Supplier<T> {
	private Class<T> type;
    public BasicSupplier(Class<T> type) {
        this.type = type;
    }
	@Override
	public T get() {
    	return type.newInstance();
    }
    public static <T> Supplier<T> create(Class<T> type) {
        return new BasicSupplier<>(type);
    }
} 
```

```java
public static void main(String[] args) {
    Stream.generate(
    	BasicSupplier.create(CountedObject.class))
    	.limit(5)
    	.forEach(System.out::println);
}
```



### Stream的中间操作

#### 筛选与切片

- 过滤：filter为过滤操作，传入Predicate类型进去test判断

```java
List<User> users = UserData.getUsers();
//获取顺序流
Stream<User> stream = users.stream();
//判断数组中的username是否存在 马
stream.filter(user->user.getUsername().contains("马")).forEach(System.out::println);
```

- 截断：只获取指定数量的数据 limit

```java
System.out.println("**************");
users.stream().limit(3).forEach(System.out::println);
```

- 跳过元素 skip

```java
users.stream().skip(3).forEach(System.out::println);
```

- 筛选：去重，通过流所生成元素的 hashCode() 和 equals() 去除重复元素所以我们的实体类要重写equals和hashcode方法

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (o == null || getClass() != o.getClass())
        return false;

    User user = (User) o;

    if (username != user.username)
        return false;
    if (password != user.password)
        return false;
    if (Double.compare(user.price, price) != 0)
        return false;
    return true;
}

@Override
public int hashCode() {
    int result;
    long temp;
    result = 17;
    result = 31 * result + (username != null ? username.hashCode() : 0);
    result = 31 * result + (username != null ? username.hashCode() : 0);
    temp = Double.doubleToLongBits(price);
    result = 31 * result + (int) (temp ^ (temp >>> 32));
    return result;
}
```

则此时输出的为u1/u3

```java
User u1 = new User("张三", "123", 1.0);
User u2 = new User("张三", "123", 1.0);
User u3 = new User("张三", "1234", 1.0);
Stream.of(u1,u2,u3).distinct().forEach(System.out::println);

```

#### 映射

- map： 将每一个元素做一系列操作后再返回

```java
//将每一个元素转化为大写
Stream.of("a","b","c").map(s -> s.toUpperCase()).forEach(System.out::println);

```

```java
//获取名称大于2的名字
Stream<String> userNames = UserData.getUsers()
        .stream()
        .map(User::getUsername);
userNames.filter(userName->userName.length()>2).forEach(System.out::println);

```

- flatmap ： 将流中的每个值都换成另一个流，然后把所有流连接成一个流

```java
@Test
public void test4() {
    //toStream会产生stream集合，Stream<Stream<Character>>类型
    // flatmap就是将stream里面的stream集合元素拿出了，组成一个新stream
    UserData.getUsers().stream().flatMap(user->SteamTest.toStream(user.getUsername())).forEach(System.out::println);
}

//产生一个有单个字符组成的stream集合
public static Stream<Character> toStream(String str){
    ArrayList<Character> list = new ArrayList<>();
    for(Character c : str.toCharArray()){
        list.add(c);
    }
    return list.stream();
}

```

#### 排序

```java
Stream.of(2, 1,  3, 4).sorted().forEach(System.out::println);

```

如果时对对象排序

```java
UserData.getUsers().stream()
        .sorted((d1,d2)->Double.compare(d1.getPrice(),d2.getPrice()))
        .forEach(System.out::println);

```

Stream的终止操作

### 匹配和查找

- allMatch(Predicate p)——检查是否匹配所有元素。

```java
//所有元素的username是否存在马
boolean t1 = UserData.getUsers()
        .stream()
        .allMatch(user -> user.getUsername().contains("马"));

```

- anyMatch(Predicate p)——检查是否至少匹配一个元素。

```java
//是否存在元素username存在马
boolean t2 = UserData.getUsers()
        .stream()
        .anyMatch(user -> user.getUsername().contains("马"));

```

- noneMatch(Predicate p)——检查是否没有匹配的元素。

```java
//是否所有元素不含有马
boolean t3 = UserData.getUsers()
        .stream()
        .noneMatch(user -> user.getUsername().contains("马"));

```

- 返回第一个元素

```java
Optional<User> first = UserData.getUsers()
        .stream()
        .findFirst();
System.out.println(first);

```

- 返回当前流中的任意元素

```java
Optional<User> any = UserData.getUsers()
        .parallelStream()
        .findAny();

```

- count——返回流中元素的总个数

```java
//计算价格大于2的数量
long count = UserData.getUsers()
        .stream()
        .filter(user -> user.getPrice() > 2.0).count();

```

- 获取最大值

```
//获取最大的价格
Optional<User> max = UserData.getUsers()
        .stream().max((u1, u2) -> Double.compare(u1.getPrice(), u2.getPrice()));

```

### 归约（reduce）

map是最元素进行映射，那么reduce可以看成时对这些映射后的元素求和

reduce(T identity, BinaryOperator)——可以将流中元素反复结合起来，得到一个值。返回 T

```java
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        Integer sum = list.stream().reduce(0, Integer::sum);
        System.out.println(sum);

```

### 收集

collect(Collector c)——将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法

```java
//返回list集合 
List<Employee> employees = EmployeeData.getEmployees();
        List<Employee> employeeList = employees.stream().filter(e -> e.getSalary() > 6000).collect(Collectors.toList());

        employeeList.forEach(System.out::println);

//返回set集合
        Set<Employee> employeeSet = employees.stream().filter(e -> e.getSalary() > 6000).collect(Collectors.toSet());

        employeeSet.forEach(System.out::println);

```

## Optional类

Optional<T> 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在

目的：为了避免空指针异常的存在

### 创建Optional类对象

```java
// 1、创建一个包装对象值为空的Optional对象
Optional<String> optStr = Optional.empty();
// 2、创建包装对象值非空的Optional对象
Optional<String> optStr1 = Optional.of("optional");
// 3、创建包装对象值允许为空的Optional对象
Optional<String> optStr2 = Optional.ofNullable(null);

```

### 判断Optional容器中是否包含对象

- boolean isPresent() : 判断是否包含对象
- void ifPresent(Consumer<? super T> consumer) ：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。

```java
public static void printName(Student student)
{
    Optional.ofNullable(student).ifPresent(u ->  System.out.println("The student name is : " + u.getName()));
}

```

### 内容处理

- ##### filter

- ##### map

- ##### flatMap

### 获取Optional容器的对象

- T get(): 如果调用对象包含值，返回该值，否则抛异常
- T orElse(T other) ：如果有值则将其返回，否则返回指定的other对象。
- T orElseGet(Supplier<? extends T> other) ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象。
- T orElseThrow(Supplier<? extends X> exceptionSupplier) ：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。



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

## 创建只读集合（不可变集合）

- java8和java9对比

```java
// java8
List<String> list1 = new ArrayList<>();
list1.add("a");
List<String> list2 = Collections.unmodifiableList(list1);
//java9
List<String> list3 = List.of("a", "b");
```

- 同事set、map等，都可以通过of来创建不可变的集合

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


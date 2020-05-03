# 特性

# Lambda表达式

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

## Lambda 表达式语法

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

## 函数式接口

如果一个接口只声明了一个接口方法，那么它就是函数式接口

## java四大函数式接口

消费型接口 Consumer<T>     void accept(T t)

供给型接口 Supplier<T>     T get()

函数型接口 Function<T,R>   R apply(T t)

断定型接口 Predicate<T>    boolean test(T t)



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

# 方法引用

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

# 构造器引用

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

# Stream

Stream 关注的是计算

Stream 自己不会存储元素。

Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。

Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行

## Stream 执行流程

 * Stream的实例化
 * 一系列的中间操作（过滤、映射、...)
 *  终止操作

说明：
 * 4.1 一个中间操作链，对数据源的数据进行处理
 * 4.2 一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用

## 创建Stream的方式

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

## Stream的中间操作

### 筛选与切片

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

### 映射

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

### 排序

```java
Stream.of(2, 1,  3, 4).sorted().forEach(System.out::println);
```

如果时对对象排序

```java
UserData.getUsers().stream()
        .sorted((d1,d2)->Double.compare(d1.getPrice(),d2.getPrice()))
        .forEach(System.out::println);
```

## Stream的终止操作

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

# Optional类

Optional<T> 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在

目的：为了避免空指针异常的存在

## 创建Optional类对象

- Optional.of(T t) : 创建一个 Optional 实例，t必须非空；
-  Optional.empty() : 创建一个空的 Optional 实例
-  Optional.ofNullable(T t)：t可以为null



## 判断Optional容器中是否包含对象
-  boolean isPresent() : 判断是否包含对象
-  void ifPresent(Consumer<? super T> consumer) ：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。

## 获取Optional容器的对象

- T get(): 如果调用对象包含值，返回该值，否则抛异常
-  T orElse(T other) ：如果有值则将其返回，否则返回指定的other对象。
- T orElseGet(Supplier<? extends T> other) ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象。
- T orElseThrow(Supplier<? extends X> exceptionSupplier) ：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。


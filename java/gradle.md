# Gradle概念

## DSL

- 领域特定语言
- 特点
  - 求专不求全，解决特定的问题

## 介绍

- 一种基于jvm的开发语言
- 结合了python，rubyde 许多强大的特性
- 可以与java完美结合，可以使用java的类库

# 环境搭建

- 下载groovy-sdk  --->https://groovy.apache.org/download.html
- 按照jvm环境
- 配置bin的环境变量path
- 查看版本

```shell
λ groovy --version
```

## idea创建groovy项目

- 新建项目，选择groovy

![](../image/java/groovy/20210303210359.png)

# Groovy语法

## 变量

- 弱类型定义(如下会自动转为java.math.BigDecimal)

```groovy
def t1 = 2.22
println t1.class;
```

- 字符串
  - 单引号：他的类型就是String类型
  - 三引号：可以换行输入，而不是字符串拼接

```groovy
def name = '''老
肖
'''
println name;
```

- - 双引号:字符串如果使用${}则自动转换：org.codehaus.groovy.runtime.GStringImpl

```groovy
def nameTmp = "我是 ${name}"
println nameTmp.class
```

## 闭包

- 有参数的闭包

```groovy
def closure = { name -> println "my name is ${name}"}
closure("老肖")
//my name is 老肖
```

- 闭包返回值

```groovy
//输出  my name is 老肖
//默认将最后一行返回
def closure = { name ->  "my name is ${name}"}
println(closure("老肖"))
```

- 闭包:阶乘

```groovy
println(fab(5))
int fab(int number) {
    int result = 1;
    //从一开始，一直执行到number，每一次调用Number的闭包
    1.upto(number, {num -> result=result*num})
    return  result;
}
```

- 累加

```groovy
//累加计算
int fab2(int number) {
    int result=0;
    //从0开始，循环number次执行闭包
    //方法最尾部参数是闭包可以写到方法外
    number.times {num -> result = result+num}
    return result;
}
```

### String 与闭包结合

- 字符串遍历

```groovy
String str = 'i have a dog';
//每个字符输出两遍
str.each {it -> print it.multiply(2)}
//输出结果
//sii  hhaavvee  aa  ddoogg
```

- 寻找符合的字符串

```groovy
String str = 'i have a 1 dog';
//寻找字符串满足的值，满足则返回对应的字符
println str.find{it -> it.isNumber()}
//输出结果
//1
```

### 三变量

- this, owner, delegate
- this: 指向本类
- owner:指向本类或者就近的闭包
- delegate:默认和owner相等，可以认为的修改（this owner 是不能修改的）

```groovy
//this == ownere == delegate
def classClouser = {
    println "this: "+ this;
    println "owner: " + owner;
    println "delegate: " + delegate;
}
classClouser.call();
```

```groovy
//this != owner ==delegate
//此刻，owner=最近的闭包的调用
def methodClouser = {
    def method = {
        println "this: "+ this;
        println "owner: " + owner;
        println "delegate: " + delegate;
    }
    method.call();
}
methodClouser.call();
```

### 委托策略

```groovy
class Student  {
    String name;
    def showName = {
        println "this def name is : ${name}"
    }
    String toString() {
        showName.call();
    }
}

class Teacher {
    String name;
}

def stu = new Student(name: "李学生");
def tea = new Teacher(name: "张老师");
//输出:this def name is : 李学生
println stu.toString();
//更改showName闭包的delegate
stu.showName.delegate = tea;
//将闭包的策略改为delegate， 此时，引用的变量为更换后的delegate
stu.showName.resolveStrategy = Closure.DELEGATE_ONLY;
//输出：this def name is : 张老师
println stu.toString();
```

## 范围

```groovy
//定义1-10的范围
def range=1..10;
//取出index=3的值
println range[3];
```

## 元编程

- 执行流程图

![](../image/java/groovy/15489991942120.png)

- 执行流程测试

```groovy
def person = new Person();
person.cry();

//会抛出MissingMethodException异常，因为cry方法不存在
```

在Person中重写invokeMethod方法，会调用此方法，

```groovy
class Person {
    @Override
    Object invokeMethod(String name, Object args) {
        println "invokeMethod: ${name}, ${args}";
    }
}
```

在person中定义methodMissing方法，智慧调用missing方法

```groovy
class Person {
    @Override
    Object invokeMethod(String name, Object args) {
        println "invokeMethod: ${name}, ${args}";
    }

    def methodMissing(String name, Object args) {
        println "methodMissing: ${name}, ${args}";
    }
}
```

- 动态添加属性
  - 输出 变态，因为已经动态的给Person添加了不存在的属性

```groovy
Person.metaClass.sex="变态";
def person = new Person();
println person.sex;U
```

- 动态添加方法

```groovy
Person.metaClass.sex="变态";

Person.metaClass.getSexUpper = {
    -> println "get Sex uppder ${sex}"
}
def person2 = new Person();
person2.getSexUpper();
```

- 动态添加静态方法

```groovy
Person.metaClass.static.getSexUpperStatic = {
    -> println "get Sex getSexUpperStatic"
}
Person.getSexUpperStatic();
```

- 动态添加全局有效
  - 默认情况下，只在当前闭包有效，想要全局有效则需要启动配置

```groovy
ExpandoMetaClass.enableGlobally()
```

## xml操作

### 解析

```groovy
def parser = new XmlParser().parseText(xml);
def to = parser.to.text();
println to;
```

## 生成xml

- 通过MarkupBuilder

# Gradle生命周期

## grade组成

- groovy核心语法
- build script block
- gradle api

## 生命周期

- Initaliztion
  - 解析整个工程中所有的project

- configuration
  - 解析所有project对象的task，构建task拓扑图
- Execution执行
  - 执行具体的task集齐依赖task

## 代码部分

- settings文件输出

```groovy
println '初始化阶段执行'
```

- build文件输出

```java
this.beforeEvaluate {
    println '初始化阶段执行'
}

this.afterEvaluate {
    println '配置阶段完成后的监听'
}

this.gradle.buildFinished {
    println '执行任务完成'
}
```

- 执行输出

```groovy
$ ./gradlew clean
初始化阶段执行

> Configure project :
配置阶段完成后的监听
执行任务完成

```



# Project

- 只要有build.gradle就是一个project

```shell
## 查看有多少个project
$ ./gradlew project
```

- 获取所有的project，在build编写

```groovy
this.getAllprojects().eachWithIndex{ Project entry, int i ->
    println "==>project : ${entry.name}"
}
```

- 对指定模块操作

```groovy
/**
 * 对指定路径的project进行操作
 */
project("child") { Project project ->
    //指定child的版本和group
    group 'org.example1'
    version '1.0-SNAPSHOT'
}
//输出：org.example1
println project("child").group;
```

- 指定所有的模块的公共部分

```groovy
allprojects {
    group 'org.example2'
    version '1.0-SNAPSHOT'
}
println project("child").group;
```

- 对所有子模块进行操作

```groovy
subprojects {
    group 'org.example3'
    version '1.0-SNAPSHOT'
}
```

## 属性

```groovy
//默认的编译文件
String DEFAULT_BUILD_FILE = "build.gradle";
String PATH_SEPARATOR = ":";
//默认输出文件夹
String DEFAULT_BUILD_DIR_NAME = "build";
String GRADLE_PROPERTIES = "gradle.properties";
```

## 自定义属性

- 根项目定义

```groovy
ext {
    def_group="org.example4"
}

subprojects {
    group this.ext.def_group
    version '1.0-SNAPSHOT'
}
```

- 使用通用文件来定义

1. 在根目录定义common.gradle文件

```groovy
//在对应的模块建立属性的键值对
ext {
    java = [
            my_group: "org.example5",
    ]
}
```

2. 在build中引入文件

```java
//apply from会自动去根目录寻找对应的文件并且引入进来
apply from: this.file("common.gradle")
```

3. 调用文件中对应的属性

```groovy
subprojects {
    group this.ext.java.my_group
    version '1.0-SNAPSHOT'
}
```

## 文件

```groovy
//获取跟路径
println getRootDir().absolutePath;
//获取build路径
println getBuildDir().absolutePath;
//获取当前模块路径
println getProjectDir().absolutePath;
```

## file

- file不需绝对路径，只需要相对于project的路径即可

```groovy
def file = file('common.gradle');
println file.text;
```

### 拷贝

- 将文件拷贝到build文件中

```groovy
copy {
    from file('common.gradle')
    into getRootProject().getBuildDir()
}
```

- 还可以拷贝文件夹

### 文件遍历

- 对src目录下进行遍历操作

```groovy
fileTree('src/') { FileTree fileTree ->
    fileTree.visit { FileTreeElement element ->
        println element.file.getName()
    }
}
```

## 依赖配置

``` 
repositories {
    mavenCentral()
    //配置远程仓库的地址
    maven {
        url 'http://maven.aliyun.com/nexus/content/groups/public/'
        //配置用户名密码
        /*credentials {
            username="xxx"
            password="xxx"
        }*/
    }
}
```

# Task

## 创建方式

```java
//通过函数创建
task helloTask() {
    println 'hellotask1'
}

//通过TaskContainer创建
this.tasks.create('hellotask2') {
    println 'hellotask2'
}
```

## 执行阶段

- 编写task

```groovy
task hellotask {
    println 'hello task1'
    //task执行前
    doFirst {
        println '==>hello task doFirst'
    }
    //task执行后
    doLast {
        println '<==hello task doLast'
    }
}
```

- 查看语句执行

```shell
$ ./gradlew hellotask

> Task :child:hellotask
==>hello task doFirst
<==hello task doLast
执行任务完成

```

- 可以利用这个来计算编译的执行时间

## task依赖

### 静态依赖

- 定义三个task
- taskZ依赖taskX,taskY

```groovy
task taskX {
    doLast {
        println 'taskX'
    }
}

task taskY {
    doLast {
        println 'taskY'
    }
}

task taskZ(dependsOn : [taskX, taskY]) {
    doLast {
        println 'taskZ'
    }
}
```

- 输出结果

```groovy
> Task :child:taskX
taskX

> Task :child:taskY
taskY

> Task :child:taskZ
taskZ
```

- 他们的执行顺序，是先执行依赖的task，再执行自己的task

### 添加依赖的另一种方式

```groovy
taskZ.dependsOn(taskX, taskY);
```

### 程序输出添加依赖

- 定义两个task

```groovy
task lib1() {
    println 'lib1'
}

task lib2() {
    println 'lib2'
}
```

- 依赖lib相关的task

```groovy
task taskZ() {
    dependsOn this.tasks.findAll {
       task -> return task.name.startsWith('lib')
    }
    doLast {
        println 'taskZ'
    }
}
```

### 指定task顺序

- 使用mustRunAfter来指定输出在xxtask之后

```groovy
task taskX {
    doLast {
        println 'taskX'
    }
}

task taskY {
    mustRunAfter(taskX)
    doLast {
        println 'taskY'
    }
}

task taskZ() {
    mustRunAfter(taskY)
    doLast {
        println 'taskZ'
    }
}
```

# SourceSet

- 可以在这个里面修改配置
- 如：给java添加资源文件

```groovy
sourceSets {
    main {
        resources.srcDirs = [
                "src/main/resources", "src/main/resources2"
        ]
    }
}
```
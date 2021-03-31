# Jackson

## 概述

1. jackson是java技术栈内最好的JSON解析工具(官网所言)
2. 除了JSON解析，jackson还是个数据处理工具集：基于流的解析库和生成库、数据绑定、数据格式化模块(Avro、XML、Protobuf、YAML等)

### 三个核心模块

- Streaming（jackson-core）：低阶API库，提供流式解析工具JsonParser，流式生成工具JsonGenerator
- Annotations（jackson-annotations）：jackson注解
- Databind (jackson-databind)：基于java对象的序列化、反序列化能力，需要前面两个模块的支持才能实现

## JsonFactory 

- jsonFactory是线程安全的（Factory instances are thread-safe and reusable after configuration (if any)）
- 引入jar包

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

### 反序列化

- JsonFactory 是jackson-core的基本功能
- JsonParser负责将JSON解析成对象的变量值，核心是循环处理JSON中的所有内容

```java
static JsonFactory jsonFactory = new JsonFactory();
/**
 * 反序列化（json -> object）
 */
private static void deserializeJSONStr() throws Exception {
    String json = "{ \"name\": \"laoxiao\", \"id\": 1 }";
    JsonParser parser = jsonFactory.createParser(json);
    if(parser.nextToken() != JsonToken.START_OBJECT) {
        parser.close();
        System.out.println("==> json 起始符号不是 {");
        return;
    }
    Person person = new Person();
    while (parser.nextToken() != JsonToken.END_OBJECT) {
        String fieldName = parser.getCurrentName();
        log.info("==>正在解析字段 [{}]", fieldName);
        parser.nextToken();
        switch (fieldName) {
            case "id": person.setId(parser.getLongValue()); break;
            case "name": person.setName(parser.getText()); break;
        }
    }
    System.out.println(person);
    parser.close();
}
```

### 序列化

- JsonGenerator负责将对象的变量写入JSON的各个属性，这里是开发者自行决定要处理哪些字段

```json
private static void serialize() throws Exception {
    Person person = new Person(1L, "laoxiao");
    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
    JsonGenerator factoryGenerator = jsonFactory.createGenerator(byteArrayOutputStream);
    factoryGenerator.useDefaultPrettyPrinter();
    factoryGenerator.writeStartObject();
    factoryGenerator.writeNumberField("id", person.getId());
    factoryGenerator.writeStringField("name", person.getName());
    factoryGenerator.writeEndObject();
    factoryGenerator.close();
    System.out.println("<==json 序列化："+ byteArrayOutputStream.toString());
}
```

## 核心API

### 常用操作

- 序列化反序列化

```java
static ObjectMapper mapper = new ObjectMapper();

private static void deserializeJSONStr() throws Exception {
    String json = mapper.writeValueAsString(new Person(1L, "laoxiao"));
    System.out.println("==> 解析的json"+json);
}

private static void serialize() throws Exception {
    String json = "{ \"name\": \"laoxiao\", \"id\": 1 }";
    Person person = mapper.readValue(json, Person.class);
    System.out.println("<==序列化的对象："+person);
}
```

### 常用配置

```java
//序列化结果格式化
mapper.enable(SerializationFeature.INDENT_OUTPUT);
//空对象不抛出异常
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
//Date、Calendar等序列化为时间格式的字符串
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
//反序列化时，遇到未知属性不要抛出异常
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
//反序列化时，空字符串对于的实例属性为null
mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
//允许字段名没有引号
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
//允许单引号
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

### JsonInclude

| 标识         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| ALWAYS       | 默认策略，任何情况都执行序列化                               |
| NON_NULL     | 非空                                                         |
| NON_ABSENT   | null的不会序列化，但如果类型是AtomicReference，依然会被序列化 |
| NON_EMPTY    | null、集合数组等没有内容、空字符串等，都不会被序列化         |
| NON_DEFAULT  | 如果字段是默认值，就不会被序列化                             |
| CUSTOM       | 此时要指定valueFilter属性，该属性对应一个类，用来自定义判断被JsonInclude修饰的字段是否序列化,在序列化的时候会执行CustomFilter的equals方法，该方法的入参就是field0的值，如果equals方法返回true，field0就不会被序列化，如果equals方法返回false时field0才会被序列化 |
| USE_DEFAULTS | 当JsonInclude在类和属性上都有时，优先使用属性上的注解，此时如果在序列化的get方法上使用了JsonInclude，并设置为USE_DEFAULTS，就会使用类注解的设置 |

### Field注解

#### JsonProperty

- jsonProperty可以作用在成员变量和方法上，作用是在**序列化**和**反序列化**操作中指定json字段的名称
- 可以作用在字段上，也可以作用在get或者set方法上

1. 序列化操作

```java
public class Person {
    private Long id;
    private String name;
    @JsonProperty(value = "file_name")
    private String fileName;
}
```

序列化后显示

```
{
  "id" : 1,
  "name" : "laoxiao",
  "file_name" : "test"
}
```

2. 反序列化

```java
String json = "{ \"name\": \"laoxiao\", \"id\": 1, \"file_name\": \"测试中\" }";
Person person = mapper.readValue(json, Person.class);
```

```
Person(id=1, name=laoxiao, fileName=测试中)

```

#### JsonIgnore

- 作用在成员变量或者方法上，指定被注解的变量或者方法不参与序列化和反序列化操作

#### JsonSerialize

- JsonSerialize用于序列化场景，被此注解修饰的字段或者get方法会被用于序列化，并且using属性指定了执行序列化操作的类

```java
public class Person {
    @JsonSerialize(using = IdJsonSerializer.class)
    private Long id;
}

```

- 可以对 对应的值调整，也可以新增字段

```java
public class IdJsonSerializer extends JsonSerializer<Long> {
    @Override
    public void serialize(Long value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        //对该字段进行
        gen.writeNumber(value+100);
        //新增一个字段
        gen.writeNumberField("id_tmp", value);
    }
}

```

```
{
  "id" : 101,
  "id_tmp" : 1,
  "name" : "laoxiao",
  "file_name" : "test"
}

```

#### JsonDeserialize

- 用于反序列化场景，被此注解修饰的字段或者set方法会被用于反序列化，并且using属性指定了执行反序列化操作的类；

```java
public class Person {
    @JsonSerialize(using = IdJsonSerializer.class)
    @JsonDeserialize(using = IdJsonDeserializer.class)
    private Long id;
    private String name;
    @JsonProperty(value = "file_name")
    private String fileName;
}

```

```java
public class IdJsonDeserializer extends JsonDeserializer<Long> {
    @Override
    public Long deserialize(JsonParser p, DeserializationContext context) throws IOException, JsonProcessingException {

        return 222L;
    }
}

```

### 方法注解

#### JsonValue

- 在序列化时起作用，可以用来注解get方法或者成员变量
- 一个类中，JsonValue只允许出现一次
- 如果注解的是方法，那么该方法的返回值就是整个实例的序列化结果；
- 如果注解的是成员变量，那么该成员变量的值就是整个实例的序列化结果；

#### JsonCreator

- 在反序列化时，当出现有参构造方法时（可能是多个有参构造方法），需要通过JsonCreator注解指定反序列化时用哪个构造方法，并且在入参处还要通过JsonProperty指定字段关系

```java
public class Person {
    private Long id;
    private String name;
    @JsonCreator
    public Person(@JsonProperty("id_field") Long id) {
		this.id = id;
    }
}

```

#### JsonAnyGetter

- 在序列化时，用Map对象的键值对转成json的字段和值

```java
public class Person {

    private Long id;

    private String name;

    private String fileName;

    private Map<String, String> map;

    @JsonAnyGetter
    private Map<String, String> getMap() {
        return map;
    }

}

```

序列化后

```json
{
  "id" : 101,
  "id_tmp" : 1,
  "name" : "laoxiao",
  "file_name" : "test",
  "aaaa" : "aaa"
}

```

#### JsonAnySetter

- 反序列化时，对json中不认识的字段，统统调用JsonAnySetter注解修饰的方法去处理
- 可以作用在成员变量上

```java
public class Person {
    private Long id;
    private String name;
    private String fileName;
    @JsonAnySetter
    private Map<String, String> map = new HashMap<>();

}

```

```shell
Person(id=222, name=laoxiao, fileName=测试中, map={tests=testsss})

```

## Spring Boot

### 配置文件

```yaml
spring:
  jackson:
    # 日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    # 序列化相关
    serialization:
      # 格式化输出
      indent_output: true
      # 忽略无法转换的对象
      fail_on_empty_beans: true
    # 反序列化相关
    deserialization:
      # 解析json时，遇到不存在的属性就忽略
      fail_on_unknown_properties: false
    # 设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    parser:
      # 允许特殊和转义符
      allow_unquoted_control_chars: true
      # 允许单引号
      allow_single_quotes: true

```

### 配置类

- 对Jackson2ObjectMapperBuilderCustomizer进行相关的处理

```java
@Configuration
public class JacksonConfig {

    @Bean
    public LocalDateTimeSerializer localDateTimeSerializer() {
        return new LocalDateTimeSerializer(J8DateUtils.GLOBAL_DATE_TIME_FORMATTER);
    }

    @Bean
    public LocalDateTimeDeserializer localDateTimeDeserializer() {
        return new LocalDateTimeDeserializer(J8DateUtils.GLOBAL_DATE_TIME_FORMATTER);
    }

    @Bean
    public LocalTimeSerializer localTimeSerializer() {
        return new LocalTimeSerializer(J8DateUtils.GLOBAL_TIME_FORMATTER);
    }

    @Bean
    public LocalTimeDeserializer localTimeDeserializer() {
        return new LocalTimeDeserializer(J8DateUtils.GLOBAL_TIME_FORMATTER);
    }

    @Bean
    public LocalDateSerializer localDateSerializer() {
        return new LocalDateSerializer(J8DateUtils.GLOBAL_DATE_FORMATTER);
    }

    @Bean
    public LocalDateDeserializer localDateDeserializer() {
        return new LocalDateDeserializer(J8DateUtils.GLOBAL_DATE_FORMATTER);
    }

    @Bean
    public DateDeserializers.DateDeserializer dateDeserializer() {
        return new DateDeserializers.DateDeserializer(DateDeserializers.DateDeserializer.instance, new SimpleDateFormat("yyyy-MM-dd"), "yyyy-MM-dd HH:mm:ss");
    }

    @Bean
    public DateSerializer dateSerializer() {
        return new DateSerializer(false, new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
    }

    @Bean
    @Primary
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return builder -> builder
            .serializerByType(LocalTime.class, localTimeSerializer())
            .deserializerByType(LocalTime.class, localTimeDeserializer())
            .serializerByType(LocalDate.class, localDateSerializer())
            .deserializerByType(LocalDate.class, localDateDeserializer())
            .serializerByType(LocalDateTime.class, localDateTimeSerializer())
            .deserializerByType(LocalDateTime.class, localDateTimeDeserializer())
            .serializerByType(Date.class, dateSerializer())
            .deserializerByType(Date.class, dateDeserializer())
            .timeZone("GMT+8")
            .dateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
    }

}

```


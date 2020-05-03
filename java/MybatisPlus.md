# 快速开始

```java
server:
##确定服务的端口，页面输入http://localhost:8081/hello/可以看到返回数据
  port: 8080

spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.94.130:3306/mytest1
    username: root
    password: 123456
```

```java
@SpringBootApplication
//扫描plus的mapper类
@MapperScan("com.xykf.quickstart")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
public interface UserMapper extends BaseMapper<User> {
}
```

```java
@Data
//该实体映射的表，如果表不一样需要配置
@TableName(value = "t_user")
public class User {
    //主键策略自增
    @TableId(type=IdType.AUTO)
    private Long id;
    @TableField //字段标示：驼峰命名，数据库是否存在等
    private String name;
    private Integer age;
    private String email;
}
```

测试类：

```java
@Test
public void testSelect() {
    System.out.println(("----- selectAll method test ------"));
    List<User> userList = userMapper.selectList(null);
    Assert.assertEquals(5, userList.size());
    userList.forEach(System.out::println);
}
```

# 全局策略

```yaml
mybatis-plus:
  global-config:
    db-config:
      idType: AUTO #主键自增
```

# 条件使用方式

## 多个字段条件查询

```java
@Test
public void testSelect() {
    System.out.println(("----- selectAll method test ------"));
    Map map = new HashMap();
    map.put("id", 1);
    QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
    userQueryWrapper.like("name", "an").or().allEq(map);
    List<User> userList = userMapper.selectList(userQueryWrapper);
    userList.forEach(System.out::println);
}
```


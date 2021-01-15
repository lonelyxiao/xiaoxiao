# 基础

## demo搭建

- 引入jar包

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 运行访问
  - 默认用户名为user
  - 密码为启动的时候的日志：Using generated security password: 57fee683-efaf-4e38-9a18-02dad9703f17

## 基本原理

- security本质是一个过滤器链

### 常见过滤器

- FilterSecurityInterceptor
  - 方法级别的权限过滤器
  - 位于过滤链的最底层

- ExceptionTranslationFilter
  - 处理认证过程中抛出的异常
- UsernamePasswordAuthenticationFilter

## 过滤器加载过程

- springboot已经通过自动装配配置了这些过滤器
- springboot通过org.springframework.web.filter.DelegatingFilterProxy来加载的过滤器的
- 项目启动后，访问连接，进入这个DelegatingFilterProxy类
- 在这个类的doFilter方法中

```java
if (delegateToUse == null) {
   WebApplicationContext wac = findWebApplicationContext();
   if (wac == null) {
      throw new IllegalStateException("No WebApplicationContext found: " +
            "no ContextLoaderListener or DispatcherServlet registered?");
   }
   delegateToUse = initDelegate(wac);
}
```

- 有一个doFilter ---> initDelegate方法
  - 在初始化方法中，从wac.getBean获取org.springframework.security.web.FilterChainProxy的过滤器

```
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
   String targetBeanName = getTargetBeanName();
   Assert.state(targetBeanName != null, "No target bean name set");
   Filter delegate = wac.getBean(targetBeanName, Filter.class);
   if (isTargetFilterLifecycle()) {
      delegate.init(getFilterConfig());
   }
   return delegate;
}
```

- FilterChainProxy --->doFilter调用doFilterInternal方法
- doFilterInternal

```java
FirewalledRequest fwRequest = firewall
				.getFirewalledRequest((HttpServletRequest) request);
HttpServletResponse fwResponse = firewall
    .getFirewalledResponse((HttpServletResponse) response);
//获取过滤器链的集合
List<Filter> filters = getFilters(fwRequest);
```

- doFilterInternal---->getFilters

## 使用自定义认证

- 创建一个类继承UsernamePasswordAuthenticationFilter
  - 重写attemptAuthentication方法
  - successfulAuthentication方法：认证成功
  - unsuccessfulAuthentication方法：认证失败
- UserDetailsService
  - 查询数据库和密码的方法写在这个接口

## 登录

### 配置文件方式

```yaml
spring:
  security:
    user:
      name: test
      password: test
```

### 配置类的方式

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String password = passwordEncoder.encode("123");
        //设置用户名test,密码123，角色为admin
        auth.inMemoryAuthentication().withUser("test").password(password).roles("admin");
    }

    /**
     * 密码加密方式
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 自定义模式

- 实现service，返回用户名密码角色
  - 当登录的时候，传入username，通过username查询出用户，返回，交给框架权限判断

```java
@Service
public class MyUserDetailsServiceImpl implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User("test", new BCryptPasswordEncoder().encode("1234"), role);
    }
}
```

# 自定义登录

- 新建登录页

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>第一个HTML页面</title>
</head>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
自定义表单验证:
<form name="f" action="/demo/login" method="post">
    <br/>
    用户名:
    <input type="text" name="username" placeholder="name"><br/>
    密码:
    <input type="password" name="password" placeholder="password"><br/>
    <input name="submit" type="submit" value="提交">
</form>
</body>
</html>
```

- 在config中配置登录事项

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
        .loginPage("/login.html")  //自定义登录页面
        .loginProcessingUrl("/demo/login") //登录提交的地址
        .successForwardUrl("/demo/login")//登录成功访问
        .failureForwardUrl("/demo/error"); //登录失败
    http.authorizeRequests()
            //不需要认证的页面
            .antMatchers("/login.html").permitAll()
            //所有请求都必须通过认证
            .anyRequest().authenticated();

    http.csrf().disable();
}
```

# 设置登录参数名

在默认登录中，必须是post请求，并且参数是username password

这是因为有默认的登录过滤器

UsernamePasswordAuthenticationFilter

```java
public class UsernamePasswordAuthenticationFilter extends
      AbstractAuthenticationProcessingFilter {
    //默认参数
   public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
   public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

   private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
   private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
   //请求方式
   private boolean postOnly = true;
```

- 如果想自定义参数名，可以在配置类如此修改

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .usernameParameter("username123")
                .passwordParameter("password123")
```

# 自定义登陆成功跳转

## 自定义跳转外链

- 查看源码

```java
.successForwardUrl("/demo/login")//登录成功访问
```

```java
public FormLoginConfigurer<H> successForwardUrl(String forwardUrl) {
   successHandler(new ForwardAuthenticationSuccessHandler(forwardUrl));
```

```java
public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
   request.getRequestDispatcher(forwardUrl).forward(request, response);
}
```

- 他其实就是实现了AuthenticationSuccessHandler进行请求装发

- 我们自己也可以实现

```java
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    private String url;

    public MyAuthenticationSuccessHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        //进行重定向
        response.sendRedirect(url);
    }
}
```

- 在配置类中配置

```
//.successForwardUrl("/demo/login")//登录成功访问
.successHandler(new MyAuthenticationSuccessHandler("www.baidu.com"))
```

##  Authentication authentication

```java
//获取权限
Collection<? extends GrantedAuthority> getAuthorities();
//获取权限 密码
Object getCredentials();
```

# 授权

## anyRequest

- 表示所有请求

- anyRequest必须在antMatchers之后（放最后），他是有顺序执行的

```java
//不需要认证的页面
.antMatchers("/login.html").permitAll()
//所有请求都必须通过认证
.anyRequest().authenticated();
```

## antMatchers

- 通配符有三种：
  ? 匹配任何单字符
  \* 匹配0或者任意数量的字符
  ** 匹配0或者更多的目录

## regexMatchers

- 正则匹配 

## 内置控制方法

```java
static final String permitAll = "permitAll";
//禁止
private static final String denyAll = "denyAll";
//匿名
private static final String anonymous = "anonymous";
//认证
private static final String authenticated = "authenticated";
// 输入了用户密码登录的
private static final String fullyAuthenticated = "fullyAuthenticated";
// 免登陆需要进入这些路径
private static final String rememberMe = "rememberMe";
```

# 权限控制

- 拥有某某权限

```java  
.antMatchers("/demo/index").hasAnyAuthority("index")
```

## 自定义403

- 定义一个异常处理handler

```java
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        PrintWriter writer = response.getWriter();
        writer.write("error");
        writer.flush();
        writer.close();
    }
}
```

- 注入到处理器中

```java
http.exceptionHandling().accessDeniedHandler(accessDeniedHandler);
```

### 基于access控制

- 上面的hasAnyAuthority本质上都是基于access表达式来控制的
- 具体有哪些表达式可以官网查询

![](..\image\java\security\20210110111144.png)

- 使用示例

```java
.antMatchers("/demo/index.html").access("hasAnyAuthority('index')")
```

## 自定义access

```java
@Component
public class MyServiceImpl implements MyService {

    @Override
    public Boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        //判断权限
        return authorities.contains(new SimpleGrantedAuthority("role1"));
    }
}
```

```java
//所有权限都走自定义认证
.anyRequest().access("@myServiceImpl.hasPermission(request, authentication)");
```

# 基于注解开发


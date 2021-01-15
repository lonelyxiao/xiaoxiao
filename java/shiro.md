# shiro中的认证

## 关键对象

- subject：主体
  - 访问系统的用户
- principal： 身份信息
  - 主体进行身份的标识，如：用户名，电话等，必须唯一
- credential：凭证信息
  - 安全信息，如密码，证书等

# 基础

## demo

- 引入jar

```java
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.5.3</version>
</dependency>
```

- 在resource下创建shiro的配置文件
  - 以.ini结尾

```
[users]
zhangsan=123
lisi=1234
```

- 认证测试

```java
public static void main(String[] args) {
    //创建安全管理器
    DefaultSecurityManager securityManager = new DefaultSecurityManager();
    //设置配置文件地址
    securityManager.setRealm(new IniRealm("classpath:shiro.ini"));
    //设置全局安全管理器
    SecurityUtils.setSecurityManager(securityManager);
    Subject subject = SecurityUtils.getSubject();
    //创建令牌，相当于登录
    UsernamePasswordToken token = new UsernamePasswordToken("zhangsan", "123");
    try {
        //登录，认证失败会抛出异常
        subject.login(token);
    } catch (UnknownAccountException e) {
        System.out.println("用户名不存在");
    } catch (IncorrectCredentialsException e) {
        System.out.println("密码错误");
    } catch (Exception e) {

    }
}
```

## login源码

- 断点在subject.login(token)处，进入方法到**org.apache.shiro.realm.SimpleAccountRealm#doGetAuthenticationInfo**方法，获取用户信息
- org.apache.shiro.authc.credential.SimpleCredentialsMatcher#doCredentialsMatch进行令牌校验

![](..\image\java\shiro\20210112230419.png)

- org.apache.shiro.realm.AuthenticatingRealm#doGetAuthenticationInfo进行认证
- org.apache.shiro.realm.AuthorizingRealm#doGetAuthorizationInfo 用来做授权

## 自定义realm

- 如果需要从数据库中查出用户，则需要自定义realm
- 实现AuthorizingRealm方法

```java
public class MyRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //授权判断判断的
        return null;
    }
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        if(token.getPrincipal().equals("zhangsan")) {
            //模拟返回用户
            return new SimpleAccount("zhangsan", "1234", this.getName());
        }
        return null;
    }
}
```

- 调用时，设置realm修改成自己的

```java
securityManager.setRealm(new MyRealm());
```

## MD5+salt

一般密码存储数据库中，是加密的，为了防止md5被穷举法计算出，所以我们使用md5+salt方式存储密码

- shiro中的md5

```java
//md5:202cb962ac59075b964b07152d234b70
Md5Hash md51 = new Md5Hash("123");
System.out.println(md51.toHex());

//md5+salt:50c6cfa137465a41726781e29d325a7a
Md5Hash md52 = new Md5Hash("123", "lonely");
System.out.println(md52.toHex());

//md5+salt+hash散列:646bea76fce01bfaec5b9a8bf36b3938
Md5Hash md53 = new Md5Hash("123", "lonely", 1024);
System.out.println(md53.toHex());
```

- md5

```java
MyRealm realm = new MyRealm();
HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
matcher.setHashAlgorithmName("md5");
realm.setCredentialsMatcher(matcher);
securityManager.setRealm(realm);
SecurityUtils.setSecurityManager(securityManager);
```

```java
if(token.getPrincipal().equals("zhangsan")) {
    //模拟返回用户
    return new SimpleAccount("zhangsan", "202cb962ac59075b964b07152d234b70", this.getName());
}
```

- md5+salt

```java
if(token.getPrincipal().equals("zhangsan")) {
    //模拟返回用户
    return new SimpleAccount("zhangsan", "50c6cfa137465a41726781e29d325a7a", ByteSource.Util.bytes("lonely"), this.getName());
}
```

- md5+salt+hash散列1024次

```java
MyRealm realm = new MyRealm();
HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
matcher.setHashAlgorithmName("md5");
matcher.setHashIterations(1024);
realm.setCredentialsMatcher(matcher);
securityManager.setRealm(realm);
SecurityUtils.setSecurityManager(securityManager);
```

# 授权

## 授权方式

- 基于角色 RBAC

```java
if(subject.hasRole("admin"))｛
    
 ｝
```



- 基于资源

```java
//对01用户有修改权限
if(syvhect.isPermission("user:update:01")) {
    
}
```

## 权限字符串

- 规则
  - 资源标识符:操作:资源实例标识
  - 如：user:create:*
  - 可以用*来表示通配符

## 授权实现

- realm中返回对应的权限

```java
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    //用户名
    System.out.println(principals.getPrimaryPrincipal());

    SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
    //添加角色
    authorizationInfo.addRole("admin");
    //添加资源权限
    authorizationInfo.addStringPermission("zhangsan:create:*");
    return authorizationInfo;
}
```

- 主体判断

```java
try {
    //登录，认证失败会抛出异常
    subject.login(token);
    //判断是否有权限
    System.out.println(subject.hasRole("admin"));;
    System.out.println(subject.isPermitted("zhangsan:create:01"));
} catch (UnknownAccountException e) {
    System.out.println("用户名不存在");
} catch (IncorrectCredentialsException e) {
    System.out.println("密码错误");
}
```

- 日志打印

```log
zhangsan
true
zhangsan
true
```


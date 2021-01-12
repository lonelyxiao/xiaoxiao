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
# 用户的相关文件

## 用户信息

### 基本信息

```shell
#查看用户信息文件 
[root@localhost ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash

```

- 第一列：
- 第二列： x ： 密码的标志，真正的密码存在/etc/shadow文件中，这个文件只有超级用户能查看
- 第三列：用户id   (系统靠这个识别用户，如果普通用户uid改成0， 则它也变成root了)
  - 0 超级用户
  - 1-499 系统用户（伪用户）
  - 500-? 普通用户
- 第四列： 组id
- 5：说明
- 6： 家目录

### 使用户不能登录

修改/etc/passwd对应用户，改成/sbin/nologin

```shell
user1:x:1000:1000::/home/user1:/sbin/nologin
```

再次登录

```shell
[root@localhost ~]# su user1
This account is currently not available.
```

## 影子文件

/etc/shadow     (密码文件)

```shell
[root@localhost home]# cat /etc/shadow
```

## 组信息

/etc/group

```shell
[root@localhost home]# cat /etc/group
root:x:0:
```

- 1: 组名
- 2：组密码为
- 3 ： 组id
- 4 ： 该组的附加用户

# 命令

## 添加用户

添加用户后，没密码的用户不允许登录的

- useradd

```shell
-G 指定附加组
-d 添加描述
```

- useradd配置文件

这个文件保存即可生效

```shell
[root@localhost ~]# vim /etc/default/useradd 

# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1  ## 密码到期时间，建议修改成0，一过期就生效
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes ## 给每个用户创建邮箱

```

```shell
[root@localhost ~]# vim /etc/login.defs 
## 密码过期时间，建议180
PASS_MAX_DAYS   99999
## 过期提醒天数， 建议7
PASS_MIN_DAYS   0
PASS_MIN_LEN    5
PASS_WARN_AGE   7

```

- userdel 

删除用户

## 修改密码

```shell
[root@localhost home]# passwd 
更改用户 root 的密码 。
新的 密码：

```

```shell
## 以管道符的方式修改密码，一般用于脚本批量新增用户
echo '123' | passwd --stdin user1

```

```shell
## 如果想用户一登录就修改密码
## 则修改密码的创建时间时间戳为0
chage -d 0 user1
```



## 修改用户信息

```shell
#添加用户的附加组

```

## 删除用户

```shell
userdel user1
## 删除用户并且删除家目录
userdel -r user1
```

## 切换用户

```shell
# -表示连带环境变量一起改变，所以，这个命令一般会加-号
su - user1
```

## 将用户添加进入组

```shell
## 将user1 添加进入group1组
gpasswd -a user1 group1
##将user1 从 group1组 中删除
gpasswd -d user1 group1
## 删除组
groupdel group1
```

## 查看用户的基本信息

```shell
id user1
```

